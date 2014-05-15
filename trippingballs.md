---
layout: page
title: Tripping Balls
comments: true
phaser: true

---

<script type="text/javascript">
    function byte2Hex(n)
    {
      var nybHexString = "0123456789ABCDEF";
      return String(nybHexString.substr((n >> 4) & 0x0F,1)) + nybHexString.substr(n & 0x0F,1);
    }

    function RGB2Color(r,g,b)
    {
      return '0x' + byte2Hex(r) + byte2Hex(g) + byte2Hex(b);
    }

    var state = {};
    state.play = true;//start playing
    var gameWidth = 600;
    var gameHeight = 600;
    var centerX = (gameWidth)/2;
    var centerY = gameHeight/2;
    var game = new Phaser.Game(gameWidth,gameHeight, Phaser.AUTO, 'contentdiv', { preload: preload, create: create, update: update });

var shape;   //init rect
var pButton;
var upButton;
var downButton;
var rButton;


function preload() {
}

var circleArray = [];

var timeForLargestRev = 1200;

function createCircleData(frequency1,frequency2,frequency3,
  phase1,phase2,phase3,
  center, width, len)
{


  while(circleArray.length > 0) {
      circleArray.pop();
  }  
  if (center == undefined) center = 128;
  if (width == undefined) width = 127;
  if (len == undefined )  len = 43;
  var noOfRev = len+1;
  var circleRad = 2;
  var circleDistance = 1+ circleRad;
  var circleRadGradient = 1.02;
  for (var i = 0; i < len; ++i)
  {
   var red = Math.sin(frequency1*i + phase1) * width + center;
   var grn = Math.sin(frequency2*i + phase2) * width + center;
   var blu = Math.sin(frequency3*i + phase3) * width + center;  
   var circleData = {};
   circleData.color = RGB2Color(red,grn,blu);
   circleData.angle = 0;
   var angleCoveredinLargestRev  = 360* noOfRev;
   var angleRate = angleCoveredinLargestRev / timeForLargestRev;

   circleData.angleRate = angleRate;
   circleData.radius = circleRad;
   circleRad *= circleRadGradient
   circleDistance += circleRad + circleData.radius;
   circleData.distance = circleDistance;
   noOfRev -=1;

   circleArray.push(circleData);

 }
}

function togglePause() {
  if(state.play) { state.play = false;}
  else { state.play = true;}
}

function resetCircles() {
  createCircleData(0.2,0.2,0.2,0,2,4)
}

var rateText;

function create() {
  upButton = game.input.keyboard.addKey(Phaser.Keyboard.UP);
  downButton = game.input.keyboard.addKey(Phaser.Keyboard.DOWN);  
  rightButton = game.input.keyboard.addKey(Phaser.Keyboard.RIGHT);  
  leftButton = game.input.keyboard.addKey(Phaser.Keyboard.LEFT);  
  pButton = game.input.keyboard.addKey(Phaser.Keyboard.P); 
  rButton = game.input.keyboard.addKey(Phaser.Keyboard.R);
  pButton.onDown.add(togglePause,this); 
  rButton.onDown.add(resetCircles,this);


  shape =  game.add.graphics(0, 0);
  createCircleData(0.2,0.2,0.2,0,2,4);
  var style = { font: "10px Arial", fill: "#ffffff" };
  rateText = game.add.text(3, 3,'rate: '+circleArray[0].angleRate, style);

}


function updateCircles(graphic)
{
  var noOfCircles = circleArray.length;
  var x,y;

  for( var i=0; i< noOfCircles ; i++) {
    var myCircleData = circleArray[i];
    myCircleData.angle += myCircleData.angleRate;
    var angleRad = Phaser.Math.degToRad(myCircleData.angle);
    x = centerX + myCircleData.distance * Math.cos(angleRad);
    y = centerY + myCircleData.distance * Math.sin(angleRad);   
    graphic.beginFill(myCircleData.color,1);
    graphic.drawCircle(x, y, myCircleData.radius);

  }
}

var revTimeSteps=200;
var revTimeGradiant=1.2;
var angleRateStep= 0.1;

function uniformIncrease() {



  if(circleArray[0].angleRate > 30) {
    console.log("speed limit reached. returning");
    return};

    var  len = 43;
    var noOfRev = len+1;
    innerCircleRate = circleArray[0].angleRate;
    innerCircleRate += angleRateStep;
    timeForLargestRev = 360 * noOfRev / innerCircleRate;
    // timeForLargestRev -= revTimeSteps;
    // timeForLargestRev /= revTimeGradiant;
    console.log("time period = "+timeForLargestRev);


    for (var i = 0; i < len; ++i)
    {


     var angleCoveredinLargestRev  = 360* noOfRev;
     var angleRate = angleCoveredinLargestRev / timeForLargestRev;

     circleArray[i].angleRate = angleRate;

     noOfRev -=1;



   }

   console.log("uniformIncrease inside rate = " + circleArray[0].angleRate);



 }

function uniformDecrease() {



  if(circleArray[0].angleRate < -30) {
    console.log("speed limit reached. returning");
    return};

    var  len = 43;
    var noOfRev = len+1;
    innerCircleRate = circleArray[0].angleRate;
    innerCircleRate -= angleRateStep;
    timeForLargestRev = 360 * noOfRev / innerCircleRate;
    // timeForLargestRev -= revTimeSteps;
    // timeForLargestRev /= revTimeGradiant;
    console.log("time period = "+timeForLargestRev);


    for (var i = 0; i < len; ++i)
    {


     var angleCoveredinLargestRev  = 360* noOfRev;
     var angleRate = angleCoveredinLargestRev / timeForLargestRev;

     circleArray[i].angleRate = angleRate;

     noOfRev -=1;



   }


   console.log("uniformDecrease inside rate = " + circleArray[0].angleRate);



 } 

var keyRateIncrement = 0.2;
function update(){


  if(state.play) {



    if (game.input.keyboard.isDown(Phaser.Keyboard.RIGHT)) { 
      uniformIncrease();
    } 
    if (game.input.keyboard.isDown(Phaser.Keyboard.LEFT)) { 
      uniformDecrease();
    }     

    if (game.input.keyboard.isDown(Phaser.Keyboard.UP)) {
      console.log("upping. angleRate = "+ circleArray[0].angleRate);
      if(circleArray[0].angleRate < 20 ) {
        for(var i=0;i<circleArray.length;i++) {
          circleArray[i].angleRate += keyRateIncrement;
        }
      }
    }

    if (game.input.keyboard.isDown(Phaser.Keyboard.DOWN)) {
      console.log("downing. angleRate = "+ circleArray[0].angleRate);
      if(circleArray[0].angleRate > -20 ) {
        for(var i=0;i<circleArray.length;i++) {
          circleArray[i].angleRate -= keyRateIncrement;
        }
      }
    }    



    shape.clear();
    updateCircles(shape);

    rateText.setText('rate: '+circleArray[0].angleRate);
  }
}

</script>


    P-Pause  
    Up/Down - Equal Rate change for all dots
    Left/Right - Gradient rate change for all dots
    R-Reset


This is the animation of [Whitney's Music Box](http://whitneymusicbox.org/). When i saw this in a gif, i couldn't resist. I had to make an interactive version to pause, slow it down, and speed it up.

There are two ways you can modify the revolution speeds. Left/Right keys gives you a uniform speed change. Up/Down is more interesting as it makes the outer/slower dots go in the reverse while the inner dots are still slowing down. 

Made using Phaser Engine (and Led Zeppelin albums I,II and III ;) ).