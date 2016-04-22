###利用Canvas和JavaScript实现五子棋
>主要技术：canvas绘制棋盘(UI)，JavaScript实现相关算法(AI)。在学习并实现五子棋功能的过程中，自己最初对算法不是特别了解，最后通过在纸上写写画画踩完了各处难点，终于完成了它的功能。感触颇多，故写篇文档分享。一来服务大众，二来加深自己理解。

####一、canvas绘制棋盘(UI)和实现落子
#####1.绘制棋盘
这是在HTML中初始化canvas：
```javascript
<canvas id="chess" width="450px" height="450px"></canvas>
```
其CSS样式：
```css
<style type="text/css">
		canvas{
			display: block;
			/*只有块级元素才可以使用下面的居中属性*/
			margin: 50px auto;
			box-shadow: -2px -2px 2px #efefef,5px 5px 5px #b9b9b9;
		}
</style>
```
①开始绘制棋盘：
```javascript
var chess = document.getElementById('chess');
var context = chess.getContext('2d');//这两句就不解释了。

var chessBoard = new Array();//定义一个数组（这里是二维的），用于保存每个位置的落子情况。
for (var i = 0; i < 15; i++) {
	chessBoard[i] = [];
	for (var j = 0; j < 15; j++) {
		chessBoard[i][j] = 0;//初始化，将棋盘所有落子位置都设置成0。
	}
}
//这里的canvas大小是450x450，棋盘按线划分规格是15+14*30+15，drawChessBoard()就完成了棋盘网格的绘制。
function drawChessBoard(){
	for (var i = 0; i < 15; i++) {
		context.moveTo(15+ i * 30,15);
		context.lineTo(15+ i * 30,435);
		context.moveTo(15,15+ i * 30);
		context.lineTo(435,15+ i * 30);
	}
	context.lineWidth = 2;
	context.strokeStyle = "#ACACAC";
	context.stroke();
}
```
②开始绘制棋子：
```javascript
var me = "true"
function oneStep(i,j,me){
	context.beginPath();
	context.arc(15+ i * 30,15+ j * 30,13,0,2*Math.PI);
	context.closePath();
	var gradient = context.createRadialGradient(15+ i * 30,15+ j * 30,0,15+ i * 30,15+ j * 30,13);//这里设置的是由前一个圆到后一个圆的渐变
	if (me) {
		gradient.addColorStop(0,'#636766');
		gradient.addColorStop(1,'#0a0a0a');//该if语句表示如果是人操作，绘制成黑色，并且实现由半径为0的白色到半径为13的黑色的圆的渐变色。
	}
	else{
		gradient.addColorStop(0,'#F9F9F9');
		gradient.addColorStop(1,'#d1d1d1');//此外表示计算机操作，颜色设置同上。
	}
	context.fillStyle = gradient;
	context.fill();
} 
```
③绘制棋盘上的背景图片：
```javascript
var logo = new Image();
logo.src = "map.png";
logo.onload = function(){
	context.drawImage(logo,50,150,353,148);//50,150表示图片绘制的起始坐标
	drawChessBoard();//注意，绘制棋盘的函数要在添加背景图片之后调用，否则背景图片会遮盖住线条。
}
```
#####2.实现落子：
```javascript
var over =false;
//点击棋盘，触发该函数
chess.onclick = function(e){//注意，这里的e在下文中表示的是onclick事件。
	if (over) {
		return;
	}
	if (!me) {
		return;
	}
	// offsetX和offsetY这两个方法，是用于取得鼠标点击的位置
	var x = e.offsetX;
	var y = e.offsetY;
	var i = Math.floor(x/30);
	var j = Math.floor(y/30);
	if (chessBoard[i][j] == 0) {//如果落子点没有已经存在的棋子
        oneStep(i,j,me);//调用绘制棋子函数
        chessBoard[i][j] = 1;//绘制完成后，将该处的位置标记为了，这样以后别的棋子就不会对该点进行覆盖了。
        
        //下面的涉及到的是AI函数调用部分，可以先忽略，先看后面。
        for(var k=0;k<count;k++){
            if (wins[i][j][k]) {
                myWin[k]++;
                comWin[k] = 6;
                if (myWin[k] == 5) {
                    window.alert("你赢了！");
                    over = true;
                }
            }
        }
        if (!over) {
            me = !me;//将下棋权限交给计算机
            comAI();//调用下方的赢法统计，注意读到该部分，记得和这里结合来读
        }
    }
}
```
####二、JavaScript实现相关逻辑算法
#####1.赢法数组
```javascript
	var wins = [];
	for(var i=0; i<15; i++){
		wins[i] = [];
		for (var j = 0; j < 15; j++) {
			wins[i][j] = [];
		}
	}//这里定义的是一个三维数组
    
	var count = 0;//定义的是赢的第几种方法
	for (var i = 0; i < 15; i++) {//i表遍历所有的行。
		for (var j = 0; j < 11; j++) {//j表示的是五个子连在一起时，首个子所在的行的位置。
			for (var k = 0; k < 5; k++) {//k用来控制是五个子连接到一块。
				wins[i][j+k][count] = true;//表示赢
			}
			count++;
		}
	}
	for (var i = 0; i < 15; i++) {//i表遍历所有的列。
		for (var j = 0; j < 11; j++) {//j表示的是五个子连在一起时，首个子所在的列的位置。
			for (var k = 0; k < 5; k++) {//k用来控制是五个子连接到一块。
				wins[j+k][i][count] = true;
			}
			count++;
		}
	}
    //下面的两个for循环，参数比较难以理解，下面是我帮助理解绘制的图。
	for (var i = 0; i < 11; i++) {
		for (var j = 0; j < 11; j++) {
			for (var k = 0; k < 5; k++) {
				wins[i+k][j+k][count] = true;
			}
			count++;
		}
	}
	for (var i = 0; i < 11; i++) {
		for (var j = 14; j > 3; j--) {
			for (var k = 0; k < 5; k++) {
				wins[i+k][j-k][count] = true;
			}
			count++;
		}
	}
```
#####2.赢法统计数组
>该部分注意结合上面的落子实现来读，在落子实现部分，onclick传入的i和j，就是这里未新定义的i和j。

```javascript
	var myWin = [];
	var comWin = [];//这里定义的是落子的个数
	for (var i = 0; i < count; i++) {
		myWin[i] = 0;//人赢的第i种方法，对应的落子的个数。
		comWin[i] = 0;//电脑赢的第i种方法，对应的落子的个数。
	}
	//电脑的策略算法函数
    //这里是对每个落子点进行记分，人分越高越需要堵，电脑的分越高，越需要五子连珠。
	var comAI = function(){
		var myScore = [];//人的子的总分，联系下文，你能了解它是二维数组。
		var max = 0;//初始化分的最大值
		var u = 0, v = 0;//初始化得分最大时，对应的落子坐标。
		var computerScore = [];//电脑的总分
        
        //下面这个for循环，主要是初始化每个落子点的初始分值
		for (var i = 0; i < 15; i++) {
			myScore[i] = [];
			computerScore[i] = [];
			for (var j = 0; j < 15; j++) {
				myScore[i][j] = 0;
				computerScore[i][j] = 0;
			}
		}
		for (var i = 0; i < 15; i++) {
			for(var j = 0; j < 15; j++){
				if (chessBoard[i][j] == 0) {//判断该落子点是否为空，即，没有被占用
					for (var k = 0; k < count; k++) {
						if (wins[i][j][k]) {//点击的坐标，对应的可以五子连珠的所有可能
							if (myWin[k] == 1) {//落子数为1，对应的分值
							myScore[i][j] += 200;
							}
							else if (myWin[k] == 2) {落子数为2，对应的分值
								myScore[i][j] += 400;
							}
							else if (myWin[k] == 3) {落子数为3，对应的分值
								myScore[i][j] += 2000;
							}
							else if (myWin[k] == 4) {落子数为4，对应的分值
								myScore[i][j] += 20000;
							}
							if (comWin[k] == 1) {//下面这些是对应的计算机的得分
								computerScore[i][j] += 220;
							}
							else if (comWin[k] == 2) {
								computerScore[i][j] += 420;
							}
							else if (comWin[k] == 3) {
								computerScore[i][j] += 2200;
							}
							else if (comWin[k] == 4) {
								computerScore[i][j] += 20000;
							}
						}
					}
				}
                //注意，这里的max是myScore和computerScore共有的最大的值
				if (myScore[i][j] > max) {//表示只要myScore大于最大值
					max = myScore[i][j];
					u = i;
					v = j;
				}
				else if(myScore[i][j] == max){
					if (computerScore[i][j] > computerScore[u][v]) {
						u = i;
						v = j;
					}
				}
				if (computerScore[i][j] > max) {
					max = computerScore[i][j];
					u = i;
					v = j;
				}
				else if(computerScore[i][j] == max){
					if (myScore[i][j] > myScore[u][v]) {
						u = i;
						v = j;
					}
				}
			}
		}
		oneStep(u,v,false);
		chessBoard[u][v] = 2;//这个数字只要不是0就行，主要为了占位
		for(var k=0;k<count;k++){
			if (wins[u][v][k]) {
				comWin[k]++;
				myWin[k] = 6;//随意的大于5 的数就行
				if (comWin[k] == 5) {
					window.alert("计算机赢了！");
					over = true;
				}
			}
		}
		if (!over) {
			me = !me;//将下棋权限交给人
		}
	}

```
