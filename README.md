# GPP
Graphics Programming Project
//Oleksandr
//g00279752
//Softwre Development 3
//Air Defence
//up arow - move gun up
//dow arrow - move down
// space -shot

<html>
<head>
	<title>Air Defence</title>
	<link rel="stylesheet" type="text/css" href="css/style.css">
</head>
<body>
	<div class="bg" id="map">
		<div class="weapon" id="weapon"></div>
		<div class="weapon-bg"></div>
		<div class="end-game" id="end-game"></div>
	</div>
</body>

<script type="text/javascript">
	
	// cheking what browser and does it suport requestAnimationFrame, if not we using function setTimeout
	if ( !window.requestAnimationFrame ) {

	    window.requestAnimationFrame = ( function() {

	        return window.webkitRequestAnimationFrame ||
	        window.mozRequestAnimationFrame ||
	        window.oRequestAnimationFrame ||
	        window.msRequestAnimationFrame ||
	        function(callback) {

	            window.setTimeout( callback, 1000 / 60 );
	        };
	    } )();
    }

	// Declaring Variables
	var map = document.getElementById('map') //obtain an element map
		, mapWidth = 900 // map width
		, mapHeight = 500 // map hight
		, isPlay = false // make sure is game is not running

		, statEl = document.getElementById('end-game') // for game statistic

		, weapon = document.getElementById('weapon') // weapon element
		, weaponW = 150 // width
		, weaponMoved = false // whether to allow movement of weapons at the moment
		, weaponMove // for moving weapon
		, weaponAngle = 0 // angel of weapon
		, weaponAngleMax = 80 // max agel of weapon
		, weaponAngleMin = 0 // min -/-

		, fireInterval = 500 //gun shoot interval
		, fire = true // is it possible to shoot
		, fireBalls = [] // an array in which add all object shots
		, fireBallSpeed = 5 // speed of a bullet
		, enemies = [] // an array in which add all object enemies
		, enemiesIntervalMin = 2000 // min time of next enemy(ms)
		, enemiesIntervalMax = 4000 // max -/-
		, enemiesSpeed = 1 // speed of enemy
		, enemyHeight = 35 // hight enemy
		, enemyWidth = 100 // width enemy
		, enemiesStartX = mapWidth //enemy appears point by Х = width of map
		, enemiesStartYMax = mapHeight-enemyHeight-20 // max point by Y(distans from top)enemy appears.= map hight
		, enemiesStartYMin = 80 // max point by Y(distans from bottom)enemy appears.= map hight 
		
		, startEnemies = false // enemy appear
		, enemyTimer //timer id for enemy
		, enemCount = 0 // number of enemy
		, nextLevel = 10 //enemy quantity to the next level
		, level = 1 // current level
		, stat = { // statistic object
			time:0, // game time
			shots:0, // shoot number
			hits:0, // number of hits 
			status: "Game Over" // game over
	};
	map.style.width = setCssValue(mapWidth); // sets the width of the map
	map.style.height = setCssValue(mapHeight); //sets the hight of the card
	
	startGame(); //game start

	// function that will run every frame of the animation
	function loop () {
		if(enemies.length) moveEnemies(); // If the array of enemies is not an empty run to move the function enemies
		if(fireBalls.length) moveFireBalls(); //If the array of shots is not an empty feature launching projectiles move
		if(weaponMoved) weaponMove(); //if enabled the movement of cannon- move the cannon gun
		stat.time += 1; //Increases the game through each frame
		if(isPlay) requestAnimationFrame(loop); // If the game is running, to start the function before the next frame loop
	}

	//start gaem function
	function startGame () {
		isPlay = true; // chenging variable for true
		startEnemies = true; //start make enemy
		document.addEventListener('keydown', startMoveWeapon, false); //event handler to the press
		document.addEventListener('keyup', stopMoveWeapon, false); // event handler to release key
		document.addEventListener('keypress', weaponFire, false); //event handler to press key
		enemyLoop(enemiesIntervalMin,enemiesIntervalMax); //run function of creating enemies and pass the minimum and maximum time
		requestAnimationFrame(loop); //start animation
	}

	//stop game function
	function stopGame () {
		isPlay = false; // stop game
		enemyStop(); //stop make enemy
	}

	//stope game function
	function gameOver() {
		stopGame(); //stop game
		showStat(); //displaying statistic
	}

	//shoe statistic function
	function showStat () {
		var percent = stat.hits / stat.shots;//get hit by dividing the percentage of hits by the number of shots
		var total = Math.floor(percent * stat.time); //get points multiply the percentage of hits in the time points
		statEl.innerHTML  = '<div class="stat-con"><div class="title">'+stat.status+'</div><div class="stat"></div><div class="row">		<span>Shots:</span><span>'+stat.shots+'</span></div><div class="row"><span>Hits:</span><span>'+stat.hits+'</span></div><div class="row"><span>Total:</span><span>'+total+'</span></div></div></div>'; //dispaying in html
		statEl.style.display = 'block';//displaying
	}

	//The constructor function of creating an enemy. returns an enemy
	function Enemy () {
		this.el = document.createElement('div'); //element

		switch(level){ //appoint a member of the class according to the current level
			case 1: this.el.className = 'enemy1';break;
			case 2: this.el.className = 'enemy2';break;
			case 3: this.el.className = 'enemy3';break;
			case 4: this.el.className = 'enemy4';break;
			case 5: this.el.className = 'enemy5';break;
		}
		
		this.X = enemiesStartX; //x cords
		this.Y = Math.floor(randomInteger(enemiesStartYMin, enemiesStartYMax));//y cords bitween min max value
		this.speed = enemiesSpeed; //enemy speed
		this.W = enemyWidth;  // width enemy
		this.H = enemyHeight; // hight
		this.el.style.left = setCssValue(this.X);  //assign coordinates css properties of the element
		this.el.style.bottom = setCssValue(this.Y); //assign coordinates css properties of the element
	}

	//makeing enemy
	function createEnemy () {
		enemCount += 1; //increase the number of enemies created
		checkLevel(); //checking level
		var enemy = new Enemy(); //creat object enemy
		map.appendChild(enemy.el); //We put the enemy on the map
		enemies.push(enemy); //add enemy to array
	}

	//drive function enemies. through the array of enemies and every move of the enemy starts the function move enemy
	function moveEnemies () {
		for (var i = 0, l = enemies.length; i < l; i++) {
			moveEnemy(enemies[i]);
		};
	}

	//move of 1 enemy
	function moveEnemy (enemy) {
		enemy.X -= enemiesSpeed; //reduce the x coordinate of the speed of the enemy
		enemy.el.style.left = setCssValue(enemy.X); //appoint a member of the enemy css left property equal to the displacement of the coordinate X. moveing
		if(enemy.X <= 0) gameOver(); //if the coordinates of the enemy is less than 0, then the enemy flew to start and stop the game
	}

	//function to create enemies at a time interval
	function enemyLoop(min,max) {
		interval = (min && max) ? randomInteger(min,max) : 0; //If you pass a minimum and maximum value randomly take from min to max, if not then 0
		if(startEnemies){ //check whether to allow the creation of enemies
			createEnemy(); // call the creation of the enemy
			enemyTimer = setTimeout(function () { //run the function of creating the enemy again after an interval of time and keep the timer id
				enemyLoop(enemiesIntervalMin,enemiesIntervalMax);
			}, interval);
		}
	}

	//stop creating enemy
	function enemyStop () {
		startEnemies = false; //stop creating enemy
		clearTimeout(enemyTimer); //clear clear the interval creating enemies by id timer
	}

	//function to delete the enemy. Gets the index of the enemy in the array of enemies
	function removeEnemy (index) {
		map.removeChild(enemies[index].el); //delete an item with the index of an array of enemies from the map
		enemies.splice(index,1); //delete the object of the enemy with the index of an array of enemies
	}

	//chek level
	function checkLevel () {
		if(enemCount > nextLevel) { //check whether the number created by the number of enemies to the next level
			nextLevel *= 2; //increas enemy amount to next level
			level +=1; //next level
			if(level >= 6){ //check max level
				stat.status = 'You Winner'; //if biger than max game over good boy
				gameOver();//stop game
				return; //end function
			}
			enemiesIntervalMax -= 1000; //to reduce the max time between the appearance of the enemy
			enemiesIntervalMin -=1000; //to reduce the min time between the appearance of the enemy
			enemiesSpeed += 0.6; //increas speed
		}
	}

	//the function of motion of projectiles. gets round
	function moveFireBall (ball) {
		ball.startOffset += fireBallSpeed; //add to offset its speed projectile
		var percent = ball.angle * 100 / 90 //We receive a percentage of the angle at which the projectile was released
			, q = 80*(percent/100)-10  //receive an offset from the left edge of the shell element according to the percentage of the angle at which it is released
			, x =  Math.round( ball.startOffset * Math.cos(degToRad(ball.angle))) + q //according to the formula getting the legs of a right triangle we obtain the coordinates plus the offset. functionality is transforms degrees to radians
			, y = Math.round( ball.startOffset * Math.sin(degToRad(ball.angle))) - q+35; //according to the formula getting the legs of a right triangle we obtain the coordinates of plus minus offset the lower edge of a cannon. functionality is transforms degrees to radians

		ball.X = x; //save cords
		ball.Y = y; //...
		ball.el.style.left = setCssValue(x); //assign element coordinate projectile
		ball.el.style.bottom = setCssValue(y); //assign element coordinate projectile
	}

	//function to check whether a shell flew over the map
	function ballLeaveMap (ball) {
		//if the left edge of the shell greater than the width card or the lower edge of the map means more height departed return true else false
		if(ball.X > mapWidth || ball.Y > mapHeight) return true; 
		return false;
	}

	//drive function is called shells at each moment of animation and enumerates the shells in the array, and applies to every function of movement
	function moveFireBalls () {
		for (var i = 0, l = fireBalls.length; i < l; i++) { //We bypass the mass of the projectile
			if(fireBalls[i]){ 
				moveFireBall(fireBalls[i]); // moving shells
				//check whether entering or leaving a shell map
				if( ballHit(fireBalls[i]) || ballLeaveMap(fireBalls[i])){ 
					// delete shel 
					removeBall (i);
					i--; //return loop to iterate back as the array is smaller by one
				}
			}
			
		};
	}

	//contact check feature. receives one shell
	function ballHit (ball) {
		for (var i = 0, l = enemies.length; i < l; i++) { //bypassing all existing enemies
			if(
				(   
					//if the right edge of the shell over the left edge of the enemy and lower right edge of the enemy or
					// The left edge of the projectile over the left edge of the enemy and the enemy is less than the right edge
					// Then they crossed horizontally
					(ball.X + ball.W >= enemies[i].X  && ball.X + ball.W <= enemies[i].X + enemies[i].W ) ||
					(ball.X >= enemies[i].X  && ball.X <= enemies[i].X + enemies[i].W ) 
				) &&
				(
					// if the upper edge of the shell above the lower edge of the enemy and the enemy is less than the upper edge or
					// The lower edge of the shell above the lower edge of the enemy and the enemy is less than the upper edge
					// Then they crossed the vertical
					(ball.Y + ball.H >= enemies[i].Y  && ball.Y + ball.H  <= enemies[i].Y + enemies[i].H ) ||
					(ball.Y >= enemies[i].Y && ball.Y <= enemies[i].Y + enemies[i].H )
				)
				
			){
				//if true - hit
				stat.hits +=1; //stat +1
				boom(enemies[i].X, enemies[i].Y); //run function explosion. We pass it the last coordinates of the enemy
				removeEnemy(i);//delet enemy
				
				return true; //return true to the previous function, remove the shell
			}			
		};
		return false; //other false
	}

	//function to create an explosion. receives the coordinates of the explosion
	function boom (x,y) {
		var el = document.createElement('div'); //make element
		el.className = 'boom'; //assign class
		el.style.left = setCssValue(x); //We put the item to the place of origin obtained
		el.style.bottom = setCssValue(y);//
		map.appendChild(el); //element to map

		//starts a timer that a second to remove an item from the map
		setTimeout(function () {
			map.removeChild(el);
		},1000);
	}

	//delete function of the projectile. gets the index
	function removeBall (index) {
		map.removeChild(fireBalls[index].el); //delete an item with the index of an array of shells from the map
		fireBalls.splice(index,1); //delete the object of the projectile with the index of an array of shells 
	}

	//The constructor function of creating a shell
	function createFireBall () {
		this.el = document.createElement('div'); //It creates an element and keep it in the returned object
		this.el.className = 'ball'; //appoint a member of the class
		this.startOffset = weaponW; //starting offset equal to the width of the projectile weapons
		this.X = 0; //X coordinate will be assigned during the movement
		this.Y = 0; //y coordinate will be assigned during the movement
		this.W = 20;  //width of the shell (such as the css property width)
		this.H = 20; //height of the projectile (such as the css property height)
		this.angle = weaponAngle; //the angle of flight of the projectile is equal to the angle of rotation of weapons at the time of the shot
	}

	//shot function. Gets a keypress events
	function weaponFire (e) {
		var charCode = getChar(e); //Get the code for the key pressed
		if(charCode == 32){ //If code 32 is a space that ignore the rest
			e.preventDefault(); //cancel the default action key
			if(!fire) return false; //If shooting is forbidden to pick off the function
			fire = false; //shoot -stop

			var ball = new createFireBall(); //make an object of shell
			map.appendChild(ball.el);//on the map
			fireBalls.push(ball); //to the array
			stat.shots +=1; //shoot stat +1

			//starts a timer which after a time interval specified in the permit change shooting
			setTimeout(function () {
				fire = true;
			},fireInterval);
		}
	}

	//function of the beginning of movement of weapons. receives an event object keydown.
	function startMoveWeapon (e) {
		var charCode = getChar(e); //Get the code for the key pressed
		if(charCode == 40 || charCode == 38){ //If the code is 40 or 38 it's the up or down ignore the rest
			e.preventDefault(); //cancel the default action key
			if(weaponMoved) return false; //if the gun spins cancel action
			//depending on the pressed key stored in the variable corresponding function
			weaponMove = (charCode == 38) ? weaponMoveUp : weaponMoveDown; 
			weaponMoved = true; //set variable true. run stored in the variable function at each point in an animation
		}
	}

	//feature stop motion weapons. receives an event object keyup.
	function stopMoveWeapon (e) {
		var charCode = getChar(e); //Get the code for the key pressed
		if(charCode == 40 || charCode == 38){ //If the code is 40 or 38 it's the up or down ignore the rest
			e.preventDefault(); //cancel the default action key
			weaponMoved = false; //set variable false. We cease to run stored in the variable function at each point in the animation
		}
	}

	//function movement of weapons up
	function weaponMoveUp () {
		weaponAngle += 1; //angel+1
		if(weaponAngle >= weaponAngleMax){ //If more than the maximum allowable degrees
			weaponAngle = weaponAngleMax; //max degree,limit to move
		} 
		weaponRotate(-weaponAngle); //run function turning weapons degrees counterclockwise
	}

	//function movement down weapons
	function weaponMoveDown () {
		weaponAngle -= 1;//angel-1
		if(weaponAngle <= weaponAngleMin){ //if the degrees less than the min
			weaponAngle = weaponAngleMin; //min degree,limit to move
		} 
		weaponRotate(-weaponAngle); //run function turning weapons degrees counterclockwise
	}

	//support functions rendered separately since they are addressed various other functions 

	// function turning weapons. receives a corner and appoint the css property rotate krosbrauzerno as browsers support this feature in different ways
	function weaponRotate (degrees) {
		weapon.style.WebkitTransform = "rotate("+degrees+"deg)";
		weapon.style.MozTransform = "rotate("+degrees+"deg)";
		weapon.style.msTransform = "rotate("+degrees+"deg)";
		weapon.style.OTransform = "rotate("+degrees+"deg)";
		weapon.style.transform = "rotate("+degrees+"deg)";
	}

	function degToRad (deg) { return deg / 180 * Math.PI; } //deg to rad
	
	//cross-browser function of receiving the key code. We accept an event object
	function getChar(event) {
		if (event.which == null) { // IE
			if (event.keyCode < 32) return null;
		  	return event.keyCode;
		}

		if (event.which != 0) {
		  	if (event.which < 32) return null;
		  	return event.which;
		}

		return null; // special characters
	}

	//random number min to max
	function randomInteger(min, max) {
	    var rand = min - 0.5 + Math.random() * (max - min + 1);
	    rand = Math.round(rand);
	    return rand;
	}

	//feature purpose css properties. It gets the value and adds it to the pixels, so as not to do it every time hands
	function setCssValue (value) {
		return value + 'px';
	}

</script>

</html>
