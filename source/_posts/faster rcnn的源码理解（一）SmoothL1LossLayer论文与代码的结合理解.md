---
title: faster rcnn的源码理解（一）SmoothL1LossLayer论文与代码的结合理解
date: "2018/09/08"
tags: ['深度学习', 'faster rcnn源码理解']
categories: ['faster rcnn', 'faster cnn源码理解']
copyright: true
---

### 源码
```c++
    // ------------------------------------------------------------------
    // Fast R-CNN
    // Copyright (c) 2015 Microsoft
    // Licensed under The MIT License [see fast-rcnn/LICENSE for details]
    // Written by Ross Girshick
    // ------------------------------------------------------------------
    
    #include "caffe/fast_rcnn_layers.hpp"
    
    namespace caffe {
    
    template <typename Dtype>
    __global__ void SmoothL1Forward(const int n, const Dtype* in, Dtype* out,
        Dtype sigma2) {
      // f(x) = 0.5 * (sigma * x)^2          if |x| < 1 / sigma / sigma
      //        |x| - 0.5 / sigma / sigma    otherwise
      CUDA_KERNEL_LOOP(index, n) {
        Dtype val = in[index];
        Dtype abs_val = abs(val);
        if (abs_val < 1.0 / sigma2) {
          out[index] = 0.5 * val * val * sigma2;
        } else {
          out[index] = abs_val - 0.5 / sigma2;
        }
      }
    }
    
    template <typename Dtype>
    void SmoothL1LossLayer<Dtype>::Forward_gpu(const vector<Blob<Dtype>*>& bottom,
        const vector<Blob<Dtype>*>& top) {
      int count = bottom[0]->count();
      caffe_gpu_sub(
          count,
          bottom[0]->gpu_data(),
          bottom[1]->gpu_data(),
          diff_.mutable_gpu_data());    // d := b0 - b1
      if (has_weights_) {
        // apply "inside" weights
        caffe_gpu_mul(
            count,
            bottom[2]->gpu_data(),
            diff_.gpu_data(),
            diff_.mutable_gpu_data());  // d := w_in * (b0 - b1)
      }
      SmoothL1Forward<Dtype><<<CAFFE_GET_BLOCKS(count), CAFFE_CUDA_NUM_THREADS>>>(
          count, diff_.gpu_data(), errors_.mutable_gpu_data(), sigma2_);
      CUDA_POST_KERNEL_CHECK;
    
      if (has_weights_) {
        // apply "outside" weights
        caffe_gpu_mul(
            count,
            bottom[3]->gpu_data(),
            errors_.gpu_data(),
            errors_.mutable_gpu_data());  // d := w_out * SmoothL1(w_in * (b0 - b1))
      }
    
      Dtype loss;
      caffe_gpu_dot(count, ones_.gpu_data(), errors_.gpu_data(), &loss);
      top[0]->mutable_cpu_data()[0] = loss / bottom[0]->num();
    }
    
    template <typename Dtype>
    __global__ void SmoothL1Backward(const int n, const Dtype* in, Dtype* out,
        Dtype sigma2) {
      // f'(x) = sigma * sigma * x         if |x| < 1 / sigma / sigma
      //       = sign(x)                   otherwise
      CUDA_KERNEL_LOOP(index, n) {
        Dtype val = in[index];
        Dtype abs_val = abs(val);
        if (abs_val < 1.0 / sigma2) {
          out[index] = sigma2 * val;
        } else {
          out[index] = (Dtype(0) < val) - (val < Dtype(0));
        }
      }
    }
    
    template <typename Dtype>
    void SmoothL1LossLayer<Dtype>::Backward_gpu(const vector<Blob<Dtype>*>& top,
        const vector<bool>& propagate_down, const vector<Blob<Dtype>*>& bottom) {
      // after forwards, diff_ holds w_in * (b0 - b1)
      int count = diff_.count();
      SmoothL1Backward<Dtype><<<CAFFE_GET_BLOCKS(count), CAFFE_CUDA_NUM_THREADS>>>(
          count, diff_.gpu_data(), diff_.mutable_gpu_data(), sigma2_);
      CUDA_POST_KERNEL_CHECK;
      for (int i = 0; i < 2; ++i) {
        if (propagate_down[i]) {
          const Dtype sign = (i == 0) ? 1 : -1;
          const Dtype alpha = sign * top[0]->cpu_diff()[0] / bottom[i]->num();
          caffe_gpu_axpby(
              count,                           // count
              alpha,                           // alpha
              diff_.gpu_data(),                // x
              Dtype(0),                        // beta
              bottom[i]->mutable_gpu_diff());  // y
          if (has_weights_) {
            // Scale by "inside" weight
            caffe_gpu_mul(
                count,
                bottom[2]->gpu_data(),
                bottom[i]->gpu_diff(),
                bottom[i]->mutable_gpu_diff());
            // Scale by "outside" weight
            caffe_gpu_mul(
                count,
                bottom[3]->gpu_data(),
                bottom[i]->gpu_diff(),
                bottom[i]->mutable_gpu_diff());
          }
        }
      }
    }
    
    INSTANTIATE_LAYER_GPU_FUNCS(SmoothL1LossLayer);
    
    }  // namespace caffe
```
### 代码讲解
SmoothL1LossLayer  计算一张图片的损失函数，对应于下图的加号右边部分

![](21.png)

i  是mini-batch的anchor的索引。
Pi  是目标的预测概率。
有物体时pi\*为1，否则为  0
ti  是一个向量，预测坐标
ti\*  是一个向量，是gt包围盒的坐标

![](22.png)  

bottom[0]  预测坐标，对应于下图的ti
bottom[1]target  坐标，对应于下图的ti\*
bottom[2]inside  有物体(fg)时为1，否则为0，对应于下图的pi\*
bottom[3]outside 没有前景（fg）也没有后景（bg）的为0，其他为1/（bg+fg），对应于加号右边的系数部分（但其实这个地方我本人还是不懂，因为论文上说的系数都是一些固定的值，如 入 =10。初始代码一直在更新，估计又换了别的方法。不论如何，在现在的代码中  outside  是乘以了后面的结果）

Lreg的公式就是下图，另x=ti - ti\*

![](23.png)

Pi × Leg(ti, ti\*)  表明只有有fg（20个物体类别）的才有回归损失

![](24.png)  

  

