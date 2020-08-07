# [Animate CC] - Hướng dẫn làm chuyển động animate khi lăn chuột (Animate on scroll)

###### tags: `AnimateCC`, `HTML5 Canvas`

Chào các bạn, hôm nay chúng ta sẽ học cách để làm chuyển động animate chạy khi chúng ta lăn chuột.

Theo mặc định, animate sẽ chạy từ đầu đến cuối giống như một video. Tuy nhiên có một số trường hợp chúng ta cần animate chạy theo thao tác người dùng, hôm nay chúng ta sẽ nghiên cứu một trường hợp như thế.

![](https://i.imgur.com/tVA2iqx.gif)

Trên đây là một animate mà mình đã làm từ trước. Các bạn có thể làm giống mình hoặc sử dụng một file animate mà bạn đã làm trước đó.

Bắt tay vào làm thôi. Trước tiên, chúng ta cần tạo một layer ở trên cùng để đón nhận chuột. Tạo một layer với tên 'Hit', sau đó vẽ một hình chữ nhật bao phủ hết màn hình.

Export thành button với tên là 'Hit'. Sau đó kéo hình chữ nhật vừa chọn vào trạng thái Hit của button (làm thế thì nó sẽ không hiển thị đè lên các layer bên dưới của mình).

![](https://i.imgur.com/5Znjwfd.png)

Ok, quay lại scene làm việc và tạo một layer Code, nhấn F9 và chúng ta bắt đầu làm thôi!

Như một thói quen, chúng ta bắt đầu với việc khai báo các thuộc tính, biến ban đầu nào. Bài hôm nay tập trung vào xử lý timeline nên chúng ta chỉ cần khai báo các thuộc tính timeline thôi. Một animate sẽ bắt đầu từ frame thứ 0.

```
var root = this;

root.targetTimeline = this;
root.targetTimeline.loop = true;
root.targetTimeline.force = 2;
root.targetTimeline.friction = 0.8;
root.targetTimeline.direction = -1; // scroll direction 
root.targetTimeline.minFrame = 0; // start frame
root.targetTimeline.maxFrame = root.targetTimeline.totalFrames - 1; // end frame
root.targetTimeline.speed = 0; // animate won't run
root.targetTimeline.pressed = false;

```

Bắt tay vào làm hàm init thôi mọi người. Bài này mục tiêu là bắt sự kiện lăn chuột vậy chúng ta hãy khai báo cho các sự kiện này trước khi triển khai nhé!
```
root.start = function () {
    createjs.Touch.enable(stage);
    root.hit.cursor = "default";
    root.gotoAndStop(root.targetTimeline.minFrame);
    canvas.addEventListener('mousewheel', root.onMouseWheel.bind(root));
    canvas.addEventListener('DOMMouseScroll', root.onMouseWheel.bind(root));
    stage.on("stagemousedown", root.onStageMouseDown.bind(root));
    createjs.Ticker.on("tick", root.tickHandler);
};

root.start();
```

Ở đây chúng ta hàm onMouseWheel cho cả 2 sự kiện 'mousewheel' và 'DOMMouseScroll'. Đó là vì trình duyệt Firefox lại nhận sự kiện DOMMouseScroll là mặc định. Và ta thì chỉ muốn scroll thực hiện hàm của chúng ta thôi, vậy nên việc khai báo 2 sự kiện này là để animate của chúng ta có thể chạy trên tất cả trình duyệt.

Giờ chúng ta cùng phân tích một chút trước khi tiếp tục nhé. Chúng ta muốn khi lăn chuột thì animate sẽ chạy phải không nào? Tức là speed (tốc độ chạy) của animate sẽ tỉ lệ với tốc độ lăn chuột của chính chúng ta (wheelDelta). 

Công thức dần hiện ra rồi phải không nào?
$$
delta = Max({-1, Min ({1, event.wheelDelta})})
$$

Tiếc là wheelDelta [không được hỗ trợ bởi một số trình duyệt](https://developer.mozilla.org/en-US/docs/Web/API/Element/mousewheel_event), thay vào đó chúng ta sử dụng thuộc tính 'detail'.

Công thức mới của chúng ta giờ sẽ là:
$$
delta = Max({-1, Min ({1, {event.wheelDelta} || {event.detail}})})
$$

Đã có công thức, giờ ta viết hàm onMouseWheel thôi.
```
root.onMouseWheel = function (e) {
    e.preventDefault();

    var evt = window.event || e;
    var delta = Math.max(-1, Math.min(1, evt.wheelDelta || -evt.detail));

    root.targetTimeline.speed = delta * root.force * root.direction;
};
```

Để xác định animate của chúng ta sẽ chạy vô hạn (loop) hay chạy một lần rồi dừng. Ta có thuộc tính 'loop' để quy định việc đó. Giờ là lúc ta định nghĩa cho nó. Nếu được lặp lại, khi chạy tới giá trị max, ta quay lại min và tiếp tục chạy. Tương tự nếu đi ngược lại. Vậy ta có 2 hàm để xử lý cho 2 trường hợp.
```
root.clamp = function(value, min, max)
{
	if (value < min)
		return min;
	
	if (value > max)
		return max;
		
	return value;
};

root.loopClamp = function(value, min, max)
{
	if (value < min)
		return max;
	
	if (value > max)
		return min;
		
	return value;
};
```

Giờ là lúc ta xử lý loop trong hàm TickHandler, hàm sẽ được gọi liên tục trong quá trình chạy.
```
root.tickHandler = function (e) {
    var clamp = root.targetTimeline.loop ? "loopClamp" : "clamp";
    var mouseY = stage.mouseY / stage.scaleY;

    if (root.targetTimeline.pressed && mouseY !== root.targetTimeline.pressedY) {
        root.targetTimeline.speed = (mouseY > root.targetTimeline.pressedY ? 1 : -1) * root.direction * root.force;
        root.targetTimeline.pressedY = mouseY;
    }
	
    root.targetTimeline.speed *= root.targetTimeline.friction;
    root.targetTimeline.gotoAndStop(root[clamp](root.targetTimeline.currentFrame + root.targetTimeline.speed, root.targetTimeline.minFrame, root.targetTimeline.maxFrame));
};
```

Vậy là xong, nhấn Ctrl+Enter và tận hưởng vuốt chuột thôi các bạn.