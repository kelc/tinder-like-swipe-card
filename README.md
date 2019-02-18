# 用CSS动画，为问卷调查设计Tinder风格的翻牌效果

/image/swipe cards.gif

最近在做一个网站项目，里面有一个功能需要用户做40分钟的问卷，其中有三个单元需要回答200多道选择题，而且答案都是二选一。为了减少答题过程的焦虑感，我在设计上做了如下取舍：
1. 在左侧和大标题上，多处提醒答题进度，让用户时时拥有掌控感；
2. 左侧进度栏，除了显示总体进度信息，也为子单元进度增加视觉元素的表达，让用户一目了然；
3. 最重要的一点，在答题区域，一次只显示一道题。

以上三点是我在过往设计问卷界面时，偏好采取的方法，但这次的案例，题目实在是太多了，而且回答方式重复度相当高，于是想为答题过程增加一些动态效果，好让过程没那么无聊。首先想到的就是堆叠卡牌的效果，结合前阵子流行的交友产品Tinder（正好问卷的对象是高中生），做类似划走卡牌的动效，应该会很合适。

要实现这个动态效果，需要用到一点点CSS动画，一点简单的JS，难度算是入门级的。对初学者来说，首先得弄清楚一个概念：CSS里面的透视属性”perspective”。

咱们的卡牌堆叠效果就是考它来实现的。如果把网页看作画布，宽和高分别对应X和Y轴的话，perspective属性就是控制Z轴的长度，也就是z-index等于0的位置，到屏幕的距离。这个距离你可以用px，rem，或者其它你喜欢的单位，在这里我们用px。如下图可以看到perspective从10px到1000px的变化，简单来说数值越小，越接近鱼眼透视效果，数值越大，越接近长焦镜头。

/image/card-perspective.png

要有上图的效果，还需要另外一个属性的配合，就是perspective-origin，在CSS里，origin就相当于物体自己的中心位置，所以perspective的origin，大家可以理解为摄像机的位置（也就是看屏幕的眼睛），在上图的三个例子里，perspective-origin都是50% 90%；两个数值分别表示X轴和Y轴的位置。你可以想象三张卡牌是从z-index为0的点开始，向着屏幕整齐排着队的三个人，perspective-origin-X小于50%，就会看到这三个人的左侧，数值越小，表示眼睛越往屏幕左边靠，于是看到三人的左侧身体就越多。50%就正好是中间。同理，perspective-origin-Y大于50%，就相当于我们从下往上看，正好就是上图效果了。

当然了，如果三个人是前胸贴后背的话，我们是看不到上图效果的，所以还需要调整两个属性让他们排队时有点距离。它们分别是transform: translate3d和z-index，对拥有perspecti属性的div来说，z-index告诉画布三者的顺序，translate3d(0, 0, 10px)会让卡牌移到需要的位置，在这里我分别让它们移动到z轴的0，-10px，-20px三个位置。之后实现动画时，只要改变translate3d的第三个数值就行，相当方便。

到了这一步，剩下的就是给三张卡牌加动画了。首先我们来分解一下文章开头的动效，其实有两个动画：一是第一张卡牌在选择答案后往右飞走；而是第一张牌飞走后，后面的卡牌移上来补充位置。

第一个动画最简单，我们只需要增加一个class，改变translate3d，rotate两个属性即可，再加上opacity和visibility，这样卡牌飞走时就既看不见，也点不着了。CSS代码如下所示：

`.card.swipe {
			transform: rotate(30deg) translate3d(120%, -50%, 0px) !important;
			opacity: 0;
			visibility: hidden;
}`
	
第二个动画复杂一些，需要用JS控制第二，第三张卡牌，改变它们translate3d的第三个值，再创建一张新的卡牌放到原来老三的位置。

我们现在来一步步看看JS的代码怎么实现吧。

首先是点击答案，控制第一张卡牌飞走并消失。

`
function pickOption() {
	const topCard = document.querySelector(".card:first-child");
	topCard.classList.add("swipe");
}
const optionBtns = document.querySelectorAll(".option");
optionBtns.forEach(optionBtn =>
	optionBtn.addEventListener("click", pickOption)
);
`

这部分并不难，在网页加载时选中两个按钮，给它们添加了响应点击事件的方法，一点击就给第一张卡牌增加swipe样式，这就触发了第一段动画。

我们希望第一张卡牌往右飞一段距离后，才开始第二段动画，所以这里要用到setTimeout()方法，这个方法有两参数，第二个是延迟多长时间，第一个是延迟时间到了以后做什么。我们往pickOption()里面添加几行代码：

`
setTimeout(function() {
	topCard.remove();
	const cards = document.querySelectorAll(".card");
	cards[0].style.transform = "translate3d(0, 0, 0px)";
	cards[0].style.zIndex = "3";
	cards[0].innerHTML = tempStr;
	cards[1].style.transform = "translate3d(0, 0, -10px)";
	cards[1].style.zIndex = "2";
`

首先，我们用remove()把第一张卡牌从DOM里删掉，然后选中剩下的卡牌，分别给它们一个新的位置和z-index值。如果卡牌比较多，你也可以用一个for循环来改变卡牌的属性。

接下来我们要新建一张卡牌，放到最末尾。

`
const cardWrap = document.querySelector(".cards-wrap");
cardWrap.insertAdjacentHTML(
	"beforeend",
	'<div class="card last-card"></div>'
);
`

insertAdjacentHTML()这个方法很有用，它第一个参数可以让你很方便的把一段HTML代码插入到指定div的开头，末尾，或者之前之后。在这里beforeend就表示cards-wrap的末尾。

到了这里，两段动效都实现了，但为了演示效果好，我们还需要给第二张卡牌添加新的问题和答案，从而实现无限点击。为了省事，我直接用JS复制第一张卡牌的内容，复制给第二张，这是实现的JS代码：

`
const topCard = document.querySelector(".card:first-child");
const tempStr = topCard.innerHTML; //复制第一张卡牌的内容
setTimeout(function() {
	…
	cards[0].innerHTML = tempStr; //复制给第二张卡牌
	…
	const newOptions = document.querySelectorAll(".option");
	newOptions.forEach(optionBtn =>
		optionBtn.addEventListener("click", pickOption)
	); //为复制的答案加上点击事件
	…
}, 300);
`

这样，我们就完成了Tinder风格的问卷答题动效了。
希望这篇不算详细的教程能帮到你，也希望以后有更多的UI设计师能用JS和CSS动画表达出自己的创意。

本教程完整的代码可以访问
[Github链接](https://github.com/kelc/tinder-like-swipe-card)
