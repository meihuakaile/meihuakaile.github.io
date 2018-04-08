---
title: 初级cnn研究辅助：python的matplotlib显示图片 之 按钮和触发事件
tags: ['python', '图片处理']
categories: ['cnn图片数据处理、显示']
copyright: true
---
一、点击显示出来的图片，出现别的：

点击左侧图片，显示右侧图片，并在你点击的位置画点。  

    
    
    from matplotlib import pyplot as py
    from matplotlib.widgets import Button,RadioButtons
    import Image
    def on_press(event):
        if event.inaxes == None:
            print "none"
            return 
        fig = event.inaxes.figure
        ax = fig.add_subplot(122)
        img_gray = Image.open("./Alex.jpg").convert("L")
        ax.imshow(img_gray, cmap="gray")
        print event.x
        ax1.scatter(event.xdata, event.ydata)
        py.axis("off")
        fig.canvas.draw()
    if __name__ == "__main__":
        img = Image.open("./Alex.jpg")
        fig = py.figure()
        fig.canvas.mpl_connect("button_press_event", on_press) 
        ax1 = fig.add_subplot(121)   
        ax1.imshow(img)
        py.axis("off")
        py.show()

效果：

![](https://img-blog.csdn.net/20160410215039789?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)  

解释：

    
    
    fig.canvas.mpl_connect("button_press_event", on_press)#在这个figure上加点击事件，点击后的情况在自己写的on_press()方法里
    def on_press(event):
           event.inaxes.figure.canvas.draw()#用于图片刷新
           event.x#事件的坐标用于其他按钮点击和figure点击发生冲突时判断返回
           event.xdata,event.ydata#鼠标点击的位置，与上面那个坐标表示形式不同

二、普通按钮

    
    
    from matplotlib import pyplot as py
    from matplotlib.widgets import Button,RadioButtons
    import Image
    def on_press(event):
        if event.inaxes == None:
            print "none"
            return 
        fig = event.inaxes.figure
        ax = fig.add_subplot(122)
        img_gray = Image.open("./Alex.jpg").convert("L")
        ax.imshow(img_gray, cmap="gray")
        print event.x
        ax1.scatter(event.xdata, event.ydata)
        py.axis("off")
        fig.canvas.draw()
    def button_press(event):
        print 'button is pressed!'
    def draw_button():
        global button#must global
        point = py.axes([0.3,0.03,0.1,0.03])
        button = Button(point, "click me")
        button.on_clicked(button_press)
    if __name__ == "__main__":
        img = Image.open("./Alex.jpg")
        fig = py.figure()
        draw_button()
        fig.canvas.mpl_connect("button_press_event", on_press) 
        ax1 = fig.add_subplot(121)   
        ax1.imshow(img)
        py.axis("off")
        py.show()

效果：

![](https://img-blog.csdn.net/20160410220718433?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)

点击按钮，控制台输出“button is pressed!”

注解：

    
    
    buttonax = plt.axes([0.8,0.03,0.05,0.03])#按钮的位置大小
    button = Button(buttonax, "i'am a button")#按钮
    button.on_clicked(save)#按钮的点击，save()是自己定议的点击后的事件
    def save(event):
        print  'i am press'
    

三、RadioButton：

    
    
    def button_press(event):
        print 'button is pressed!'
    def radio_press(label):
        print 'select: ',label
        radiobutton.set_active(0)
    def draw_button():
        global button#must global
        global radiobutton
        print 'button'
        point = py.axes([0.2,0.03,0.1,0.03])
        button = Button(point, "click me")
        button.on_clicked(button_press)
        point_two = py.axes([0.6, 0.03, 0.2, 0.05])
        radiobutton = RadioButtons(point_two, ("select me", "or me"))
        radiobutton.on_clicked(radio_press)
    
    if __name__ == "__main__":
        img = Image.open("./Alex.jpg")
        fig = py.figure()
        draw_button()
        fig.canvas.mpl_connect("button_press_event", on_press) 
        ax1 = fig.add_subplot(121)   
        ax1.imshow(img)
        py.axis("off")
        py.show()

效果：

![](https://img-blog.csdn.net/20160410223428675?watermark/2/text/aHR0cDovL2Jsb
2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravi
ty/Center)

注解：

    
    
    radiobutton.set_active(0)#"DIFFERENT"被选择
    def select(label):
          print "label"#label is "DIFFERENT" or "SAME"  

  
然后你可能会发现点击右侧按钮时也会触发右侧出现图片，这该怎么办呢？

在一的例子中有event.x就是干这个用的，通过判断这个值判断鼠标点击的位置。但是没人的电脑不同，结果不同。  

