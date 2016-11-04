var aneObj = function() {
    //x轴
    this.x = [];
    //高度
    this.len = [];

};
aneObj.prototype.num = 50;

aneObj.prototype.init = function(){
    for(var i=0;i<this.num;i++){
        this.x[i] = i * 16 + Math.random() * 20;
        this.len[i] = 200 + Math.random() * 50;
    }
};
//绘制海葵
aneObj.prototype.draw = function(){

    ctx2.save();
    ctx2.globalAlpha = 0.6;
    ctx2.lineWidth = 20;
    ctx2.lineCap = "round";
    ctx2.strokeStyle = "#3e154e";
    for(var i=0;i < this.num;i++){
        //beginPath,moveTo,lineTo,stroke,strokeStyle,lineWidth,lineCap,globalAlpha
        ctx2.beginPath();
        ctx2.moveTo(this.x[i],canHeight);
        ctx2.lineTo(this.x[i],canHeight-this.len[i]);
        ctx2.stroke();
    }
    ctx2.restore();  //一旦出去，样式就没有了
};


var babyObj = function(){
    this.x;
    this.y;
    this.angle;
    this.babyEye = new Image();
    this.babyBody = new Image();
    this.babyTail = new Image();
}
babyObj.prototype.init = function(){
    this.x = canWidth * 0.5 -50;
    this.y = canHeight * 0.5 - 50;
    this.angle = 0;
    this.babyEye.src = "./src/babyEye0.png";
    this.babyBody.src = "./src/babyFade0.png";
    this.babyTail.src = "./src/babyTail0.png";
}
babyObj.prototype.draw = function(){
    //lerp x,y
    this.x = lerpDistance(mom.x, this.x, 0.98);
    this.y = lerpDistance(mom.y, this.y, 0.98);

    //lerp angle
    var deltaY = mom.y -this.y;
    var deltaX = mom.x -this.x;
    var beta = Math.atan2(deltaY,deltaX) + Math.PI;  //大鱼的角度 -PI~PI之间


    this.angle = lerpAngle(beta,this.angle,0.6);


    //ctx1
    //只有在这个区间才有效
    ctx1.save();
    //translate
    ctx1.translate(this.x,this.y);
    ctx1.rotate(this.angle);
    ctx1.drawImage(this.babyTail,-this.babyTail.width * 0.5+23,-this.babyTail.height * 0.5);
    ctx1.drawImage(this.babyBody,-this.babyBody.width * 0.5,-this.babyBody.height * 0.5);
    ctx1.drawImage(this.babyEye,-this.babyEye.width * 0.5,-this.babyEye.height * 0.5);
    ctx1.restore();
}
//background
function drawBackground(){
    ctx2.drawImage(bgPic,0,0,canWidth,canHeight);
}

//判断大鱼和果实的距离
function momFruitsCollision(){
    for(var i=0;i< fruit.num;i++){
        if(fruit.alive[i]){
            var l = calLength2(fruit.x[i],fruit.y[i],mom.x,mom.y);
            if(l < 900){
                //fruit eaten
                fruit.dead(i);
            }
        }
    }
}

//游戏规则：保持屏幕上有15个果实

var fruitObj = function(){
     this.alive = []; //bool
     this.x = [];
     this.y = [];
     this.l = [];  //图片的长度
     this.spd = [];
     this.fruitType = [];
     this.orange = new Image();
     this.blue = new Image();
 };
 //果实总个数
 fruitObj.prototype.num = 30;
 //初始化
fruitObj.prototype.init = function(){
    for(var i=0;i<this.num;i++){

        this.alive[i] = false;
        this.x[i] = 0;
        this.y[i] = 0;
        this.l[i] = 0;
        this.spd[i] = Math.random()*0.017 + 0.003 ;  //[0.003 0.02)
        this.fruitType[i] = "";
        //this.born(i);  //让所有的果实都出生
    }
    this.orange.src = "./src/fruit.png";
    this.blue.src = "./src/blue.png";
};
 //绘制海葵产出的果实
fruitObj.prototype.draw = function(){
     for(var i=0;i<this.num;i++){
        //draw
         //find an ane, grow,fly up
         if(this.alive[i]){
             if(this.fruitType[i] == "blue"){
                var pic = this.blue;
             }
             else {
                 var pic = this.orange;
             }
             if(this.l[i] <= 14){
                 this.l[i] += this.spd[i] * deltaTime;//让过程变得平缓
             }
             //果实向上浮，就是y轴的位置在减少
             else{
                 this.y[i] -= this.spd[i] * 7 * deltaTime;
             }
             //console.log(this.l[i]);
             ctx2.drawImage(pic, this.x[i] - this.l[i] * 0.5, this.y[i] - this.l[i] * 0.5, this.l[i], this.l[i]);
             if(this.y[i] < -10){
                 this.alive[i] = false;
             }
         }
    }
};
//fruitObj.prototype.update = function(){
//    var num = 0;
//    for(var i=0;i < this.num;i++){
//        if(this.alive[i]){
//            num++;
//        }
//    }
//}
 //果实出生
fruitObj.prototype.born = function(i){
    var aneID = Math.floor(Math.random()*ane.num);  //0~49
    this.x[i]=ane.x[aneID];   //果实出生的位置
    this.y[i]= canHeight-ane.len[aneID];   //canvas的高度-海葵的高度
    this.l[i] = 0;
    this.alive[i] = true;
    var ran  = Math.random();
    if(ran < 0.2){
        this.fruitType[i] = "blue";
    }
    else{
        this.fruitType[i] = "orange";   //orange blue
    }

}

fruitObj.prototype.dead = function(i){
    this.alive[i] = false;
}

//果实监视功能
function fruitMonitor(){
    var num = 0;
    for(var i=0;i<fruit.num;i++){
        if(fruit.alive[i]){
            num++;
        }
    }
    if(num<15){
        //send fruit
        sendFruit();
        return ;
    }
}
function sendFruit(){
    for(var i=0;i<fruit.num;i++){
        if(!fruit.alive[i]){
            fruit.born(i);
            return;
        }
    }
}
var can1;
var can2;

var ctx1;
var ctx2;

var canWidth;
var canHeight;

//帧与帧之间的时间
//lastTime是上一帧的时间
var lastTime;
//deltaTime是下一帧的时间
var deltaTime;

var bgPic = new Image();

var ane;
var fruit;

var mom;
var baby;

var mx;
var my;

window.onload = game;

function game(){
    init();

    lastTime = Date.now();
    deltaTime = 0;
    gameloop();
}
function init(){
    //获得canvas context
    can1 = document.getElementById('canvas1');  //前景 fishes,dust,UI,circle
    ctx1 = can1.getContext('2d');
    can2 = document.getElementById('canvas2');  //背景  background,ane,fruits
    ctx2 = can2.getContext('2d');

    can1.addEventListener('mousemove',onMouseMove,false);

    bgPic.src = "./src/background.jpg";
    canWidth = can1.width;
    canHeight = can1.height;

    ane = new aneObj();
    ane.init();

    fruit = new fruitObj();
    fruit.init();

    mom = new momObj();
    mom.init();

    baby = new babyObj();
    baby.init();

    mx = canWidth * 0.5;
    my =  canHeight * 0.5;

}

function gameloop(){
    window.requestAnimFrame(gameloop);
    var now = Date.now();
    deltaTime = now-lastTime;   //每两帧之间的间隔
    lastTime = now;
    //页面切换后，果实不会受延迟的影响而变大
    if(deltaTime>40){
        deltaTime=40;
    }

    drawBackground();
    ane.draw();
    fruitMonitor();
    fruit.draw();
    //需要清空画布
    ctx1.clearRect(0,0,canWidth,canHeight);
    mom.draw();
    momFruitsCollision();

    baby.draw();

}

function onMouseMove(e){
    if(e.offsetX || e.layerX){
        mx = e.offsetX == undefined ? e.layerX :e.offsetX;
        my = e.offsetY == undefined ? e.layerY :e.offsetY;
    }
}

var momObj = function(){
    this.x;
    this.y;
    this.angle;
    this.bigEye = new Image();
    this.bigBody = new Image();
    this.bigTail = new Image();  //绘制大鱼的尾巴
}
momObj.prototype.init = function(){
    this.x=canWidth * 0.5;
    this.y=canHeight * 0.5;
    this.angle = 0;
    this.bigEye.src = "./src/bigEye0.png";
    this.bigBody.src = "./src/bigSwim0.png";
    this.bigTail.src = "./src/bigTail0.png";
}
momObj.prototype.draw = function(){
    //lerp x,y
    this.x = lerpDistance(mx, this.x, 0.98);
    this.y = lerpDistance(my, this.y, 0.98);

    //delta angle
    //Math.atan2(x,y);
    var deltaY = my -this.y;
    var deltaX = mx -this.x;
    var beta = Math.atan2(deltaY,deltaX) + Math.PI;  //大鱼的角度 -PI~PI之间

    //lerp angle
    this.angle = lerpAngle(beta,this.angle,0.6);

    //ctx2
    ctx1.save();
    ctx1.translate(this.x,this.y);
    ctx1.rotate(this.angle);
    ctx1.drawImage(this.bigTail,-this.bigTail.width * 0.5+30,-this.bigTail.height * 0.5);
    ctx1.drawImage(this.bigBody,-this.bigBody.width * 0.5,-this.bigBody.height * 0.5);
    ctx1.drawImage(this.bigEye,-this.bigEye.width * 0.5,-this.bigEye.height * 0.5);

    ctx1.restore();
}





