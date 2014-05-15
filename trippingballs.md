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

var gameWidth = 800;
var gameHeight = 800;
var game = new Phaser.Game(gameWidth,gameHeight, Phaser.AUTO, 'contentdiv', { preload: preload, create: create, update: update });

var shape   //init rect

function preload() {
}

function create() {
	shape =  game.add.graphics(0, 0);
	// shape.lineStyle(2, 0x0000FF, 1); // width, color (0x0000FF), alpha (0 -> 1) // required settings
	// shape.beginFill(0xFFFF0B, 1);
	// shape.drawCircle(40, 100, 50);
	var frequency = 0.3;
	var amplitude = 50;
	var center = 50;
	var samples = 500;

	for (var i = 0; i < samples; ++i)
	{
	   v = Math.sin(frequency*i) * amplitude + center;
	   console.log(v);
	   // Note that &#9608; is a unicode character that makes a solid block
	   // shape.beginFill(RGB2Color(v,v,v),1);
	   // shape.drawCircle(40+10*i,100,5);
	   shape.beginFill(0xFFFFFF,1);
	   shape.drawRect(2+5*i+i, 10, 1, v); // (x, y, w, h) 
	}	
}

function update(){
	
}

</script>
