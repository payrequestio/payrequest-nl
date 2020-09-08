---
title: PayRequest Game
tags: [color-logo]
layout: none
---

<canvas id="gamescreen" width="960" height="640"></canvas>

<!--images-->
<img id = "player" src="https://payrequest.io/assets/logos/Icon%20white.svg" style="display: none"/>

<img id = "smoke" src="http://www.clker.com/cliparts/L/m/5/L/7/N/grey-smoke-hi.png" style="display: none"/>

<img id = "obstacles" src="http://opengameart.org/sites/default/files/styles/medium/public/metal%20plate%20tex.png.preview.jpg" style="display: none"/>


<style>
	canvas{
  background: #11111b;
}
</style>

<script>
/*JS canvas game by Guthrie Schoolar*/
"use strict";

//get context of canvas to manipulate
var canvas = document.getElementById("gamescreen");
var ctx = canvas.getContext("2d");

var keyDownHandler;
var keyUpHandler;

var rand = function(max, min) {
		return Math.floor(Math.random() * (max + min + 1) + min);
};//rand
	
//object variables
var player = { 
	lives: 3,
	score: 0,
	img: new Image(),
	smoke: new Image(),
	xPos: canvas.width / 6,
	yPos: canvas.height / 2,
	vel: 5,
	draw: function() {
		ctx.drawImage(player.img, player.xPos, player.yPos, 50, 50);
	}
  
};

var obsArray = new Array();
var obstacle = {
	width: 64,
	height: canvas.height,
	rand: rand(-canvas.height + 200, -128),
	xPos: 500,
	yPos: 100,
	vel: 3,
	img: new Image(),
	pat: 0,
	init: function() {
		//adjust offset
		for (var i = 0; i < 3; i += 1) {
			var o = Object.assign({}, obstacle);
			obsArray[i] = o;
			obsArray[i].rand = rand(-canvas.height + 200, -75);
			obsArray[i].xPos = canvas.width + 350 * (1 + i);
		}
	},
	draw: function(num) {
		for (var i = 0; i < 3; i += 1) {
			obsArray[i].xPos -= obstacle.vel;
			obsArray[i].yPos =  obsArray[i].rand;
			//check for end of path
			if (obsArray[i].xPos < -128) {
				obsArray[i].rand = rand(-canvas.height + 200, -75);
				obsArray[i].xPos = canvas.width;
			}
			//actual draw part
			ctx.beginPath();
			ctx.drawImage(obstacles.img, obsArray[i].xPos, obsArray[i].yPos, obsArray[i].width, obsArray[i].height);
			ctx.fillStyle = obstacles.pat;
			ctx.fill();
			ctx.drawImage(obstacles.img, obsArray[i].xPos, obsArray[i].yPos + canvas.height + 128, obsArray[i].width, obsArray[i].height);
			ctx.fill();
			ctx.closePath();
		}
	}
};

//gamestates and HUD
var game = {
	state: "game",
	drawHUD: function() {
		ctx.beginPath();
		ctx.rect(0, 0, canvas.width, 75);
		ctx.fillStyle = "#bcbcbc";
		ctx.fill();
		
		//text
		ctx.font = "20px Arial";
		ctx.fillStyle = "black";
	    ctx.fillText("PayRequest Space Game", 20, 30);
		ctx.fillText("Score: " + player.score.toFixed(0), canvas.width - 200, 30);
		ctx.closePath();
	},
	check: function() {
		if (game.state === "game") {
			return;
		}//if
		
		if (game.state === "lose") {
			//draw the lose frame
			ctx.rect(canvas.width / 4, canvas.height / 4, canvas.width / 2, canvas.height / 2);
			ctx.fillStyle = "#bdbdbd";
			ctx.fill();
			
			ctx.font = "36px Arial";
			ctx.fillStyle = "black";
			ctx.fillText("GAME OVER", canvas.width / 2 - 110, canvas.height / 4 + 80);
			ctx.fillText("Score: " + player.score.toFixed(0), canvas.width / 2 - 80, canvas.height / 2);
			ctx.fillText("Press Enter to retry", canvas.width / 2 - 150, canvas.height / 2 + 80);
		}//if
		
		else {
			game.state = "game"; 
		}
	},
	
	reset: function() {
		//player reset
		player.score = 0;
		player.lives = 3;
		player.vel = 5;
		player.xPos = canvas.width / 6;
		player.yPos = canvas.height / 2;
		
		//obstacle reset
		obstacle.init();
		obstacle.vel = 3;
		game.state = "game";
		return;
	},
}

//image handler

var array = new Array();
var ctr = 0;
var stars = {
	ctr: 0,
	randFactor: 1,
	width: 1,
	height: 1,
	xPos: canvas.width,
	yPos: Math.random() * canvas.height,
	vel: .5,
	draw: function() {
		var s = Object.assign({}, stars);
		array[ctr] = s;

		//randomizer for parallax stars!
		array[ctr].randFactor = rand(1, 3);
		array[ctr].yPos = Math.random() * canvas.height;
		array[ctr].width *= array[ctr].randFactor;
		array[ctr].height *= array[ctr].randFactor;

		for (var i = 0; i < ctr; i += 1) {
			if (array[i].xPos < 5) {
				continue; //do not show if drawn beyond canvas
			}//if
			array[i].xPos -= array[i].vel * array[i].randFactor;//incrememnt position
			//draw
			ctx.beginPath();
			ctx.rect(array[i].xPos, array[i].yPos, array[i].width, array[i].height);
			ctx.fillStyle = "rgba(255,255,255,1)";
			ctx.fill();
			ctx.closePath();
		}//for
		//only add to the array every three times
		stars.ctr += 1;
		if (stars.ctr % 3 === 0) {
			ctr += 1;
		}//if
	}
};

function loadImages() {
		player.img = document.getElementById("player");
		obstacles.img = document.getElementById("obstacles");
		obstacles.pat = ctx.createPattern(obstacles.img, "repeat");
}//loadImages

//keyboard handler
var keyboard = {
	left: false,
	right: false,
	up: false,
	down: false,
	downHandler: function(e) {
		switch (e.keyCode) {
			case 37:
				keyboard.left = true;
				break;
			case 38:
				keyboard.up = true;
				break;
			case 39: 
				keyboard.right = true;
				break;
			case 40:
				keyboard.down = true;
				break;
			case 13:
				game.reset();
				game.state = "game";
				break;
		}//switch
	},//function
	
	upHandler: function(e) {
		switch (e.keyCode) {
			case 37:
				keyboard.left = false;
				break;
			case 38:
				keyboard.up = false;
				break;
			case 39:
				keyboard.right = false;
				break;
			case 40:
				keyboard.down = false;
				break;	
		}//switch
	},//function
};//keyboard
//event listeners for the document
document.addEventListener("keydown", keyboard.downHandler, false);
document.addEventListener("keyup", keyboard.upHandler, false);

//physics functions
function move() {
	if(keyboard.left && player.xPos > 5) {
		player.xPos -= player.vel;
	}
	if (keyboard.right && player.xPos < canvas.width) {
		player.xPos += player.vel;
	}
	if (keyboard.up && player.yPos > 75) {
		player.yPos -= player.vel;
	}
	if (keyboard.down && player.yPos < canvas.height - 60) {
		player.yPos += player.vel;
	}
}//move

function collision() {
	for (var i = 0; i < 3; i += 1) {
		if (player.xPos - obsArray[i].xPos < 5 && player.xPos - obsArray[i].xPos > -2) {
			if (player.yPos < obsArray[i].yPos + canvas.height + 128 && player.yPos > obsArray[i].yPos + obsArray[i].height) { 
				player.score += 100;
			}//if
			
			else {
				game.state = "lose";
			}//else
		}//if
	}//for
}//collision

//main function
function draw() {
	stars.draw();
	if (game.state === "game") {
		player.draw();
		obstacle.draw();
	}//if
	game.drawHUD();
}
 
Window.Onload = function() {
	loadImages();
			   }
Window.Onload();
obstacle.init(); //initialize the obstacles
function main() {
  	//erase the rectangle each frame
	ctx.clearRect(0, 0, canvas.width, canvas.height);
	game.check(); //check the gamestate
	draw();
	
	if(game.state === "game") {
		collision();
		move();
		player.score += .1;
		if (player.score.toFixed(0) % 1000 === 0) {
			obstacle.vel += .005;
			player.vel += .0075;
		}
	}//if
}
setInterval(main, 10);
</script>