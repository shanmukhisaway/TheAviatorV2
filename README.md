# The Aviator 2

Updated version of Kaim Maaloul's The Aviator, see his [GitHub](https://github.com/yakudoo/TheAviator).

## Start

Clone repository, in the code directory run `php -S localhost:8123` and in your browser visit [http://localhost:8123/](http://localhost:8123/).

## License

Integrate or build upon it for free in your personal or commercial projects. Don't republish, redistribute or sell "as-is".

<h2 class="wp-block-heading">What makes a game&nbsp;fun?</h2>



<p>While there obviously is no definitive recipe there are a few key mechanics that will maximize your chances of generating fun. There is a great compilation on <a href="https://www.gamedesigning.org/gaming/great-games/" rel="noreferrer noopener" target="_blank">gamedesigning.org</a>, so let’s see which items apply already:</p>



<p>✅ <strong>Great controls<br></strong>✅ <strong>An interesting theme and visual style<br></strong>🚫 <strong>Excellent sound and music<br></strong>🚫 <strong>Captivating worlds<br></strong>🤔 <strong>Fun gameplay<br></strong>🚫 <strong>Solid level design<br></strong>🚫 <strong>An entertaining story &amp; memorable characters<br></strong>🤔 <strong>Good balance of challenge and reward<br></strong>✅ <strong>Something different</strong></p>



<p>We can see there’s lots to do, too much for a single article of course, so we will get to the general game layout, story, characters and balance later. Now we will improve the gameplay and add sounds — let&#8217;s go!</p>



<h2 class="wp-block-heading">Adding weapons</h2>



<p>Guns are always fun! Some games like Space Invaders consist entirely of shooting and it is a great mechanic to add visual excitement, cool sound effects and an extra dimension to the skill requirement so we not only have the up and down movement of the aircraft.</p>



<p>Let’s try some simple gun designs:</p>



<div class="wp-block-image"><figure class="aligncenter is-resized"><img decoding="async" src="https://cdn-images-1.medium.com/max/1600/1*o5seoFWkphDF2ezGRgs1ww.png" alt="" width="521" height="216"/>
<br/>  
<figcaption>The “Simple gun” (top) and the “Better gun” (bottom).</figcaption></figure></div>
<br/>
<p>These 3D models consist of only 2–3 cylinders of shiny metal material:</p>



<pre class="wp-block-code has-small-font-size"><code>const metalMaterial = new THREE.MeshStandardMaterial({
    color: 0x222222,
    flatShading: true,
    roughness: 0.5,
    metalness: 1.0
})

class SimpleGun {
    static createMesh() {
        const BODY_RADIUS = 3
        const BODY_LENGTH = 20

        const full = new THREE.Group()
        const body = new THREE.Mesh(
            new THREE.CylinderGeometry(BODY_RADIUS, BODY_RADIUS, BODY_LENGTH),
            metalMaterial,
        )
        body.rotation.z = Math.PI/2
        full.add(body)

        const barrel = new THREE.Mesh(
            new THREE.CylinderGeometry(BODY_RADIUS/2, BODY_RADIUS/2, BODY_LENGTH),
            metalMaterial,
        )
        barrel.rotation.z = Math.PI/2
        barrel.position.x = BODY_LENGTH
        full.add(barrel)

        return full
    }
}</code></pre>



<p>We will have 3 guns: A <em>SimpleGun</em>, then the <em>DoubleGun</em> as just two of those and then the <em>BetterGun </em>which has just a bit different proportions and another cylinder at the tip.</p>



<div class="wp-block-image"><figure class="aligncenter"><img decoding="async" src="https://cdn-images-1.medium.com/max/1600/1*FHDJLF-2Bs0dqlaIC_IqMQ.png" alt=""/>
<br/>
<figcaption>Guns mounted to the&nbsp;airplane</figcaption></figure></div>
<br/>
<p>Positioning the guns on the plane was done by simply experimenting with the positional x/y/z values.</p>



<p>The shooting mechanic itself is straight forward:</p>



<pre class="wp-block-code has-small-font-size"><code>class SimpleGun {
  downtime() {
    return 0.1
  }

  damage() {
    return 1
  }

  shoot(direction) {
    const BULLET_SPEED = 0.5
    const RECOIL_DISTANCE = 4
    const RECOIL_DURATION = this.downtime() / 1.5

    const position = new THREE.Vector3()
    this.mesh.getWorldPosition(position)
    position.add(new THREE.Vector3(5, 0, 0))
    spawnProjectile(this.damage(), position, direction, BULLET_SPEED, 0.3, 3)

    // Little explosion at exhaust
    spawnParticles(position.clone().add(new THREE.Vector3(2,0,0)), 1, Colors.orange, 0.2)

    // Recoil of gun
    const initialX = this.mesh.position.x
    TweenMax.to(this.mesh.position, {
      duration: RECOIL_DURATION,
      x: initialX - RECOIL_DISTANCE,
      onComplete: () =&gt; {
        TweenMax.to(this.mesh.position, {
          duration: RECOIL_DURATION,
          x: initialX,
        })
      },
    })
  }
}

class Airplane {
  shoot() {
    if (!this.weapon) {
      return
    }

    // rate-limit shooting
    const nowTime = new Date().getTime() / 1000
    if (nowTime-this.lastShot &lt; this.weapon.downtime()) {
      return
    }
    this.lastShot = nowTime

    // fire the shot
    let direction = new THREE.Vector3(10, 0, 0)
    direction.applyEuler(airplane.mesh.rotation)
    this.weapon.shoot(direction)

    // recoil airplane
    const recoilForce = this.weapon.damage()
    TweenMax.to(this.mesh.position, {
      duration: 0.05,
      x: this.mesh.position.x - recoilForce,
    })
  }
}

// in the main loop
if (mouseDown&#091;0] || keysDown&#091;&#039;Space&#039;]) {
  airplane.shoot()
}</code></pre>



<p>Now the collision detection with the enemies, we just check whether the enemy’s bounding box intersects with the bullet’s box:</p>



<pre class="wp-block-code has-small-font-size"><code>class Enemy {
	tick(deltaTime) {
		...
		const thisAabb = new THREE.Box3().setFromObject(this.mesh)
		for (const projectile of allProjectiles) {
			const projectileAabb = new THREE.Box3().setFromObject(projectile.mesh)
			if (thisAabb.intersectsBox(projectileAabb)) {
				spawnParticles(projectile.mesh.position.clone(), 5, Colors.brownDark, 1)
				projectile.remove()
				this.hitpoints -= projectile.damage
			}
		}
		if (this.hitpoints &lt;= 0) {
			this.explode()
		}
	}

	explode() {
		spawnParticles(this.mesh.position.clone(), 15, Colors.red, 3)
		sceneManager.remove(this)
	}
}</code></pre>



<p>Et voilá, we can shoot with different weapons and it’s super fun!</p>



<figure class="wp-block-image"><img decoding="async" src="https://cdn-images-1.medium.com/max/1600/1*auawAUzJkfAho1opTT94Mg.gif" alt=""/></figure>



<p></p>



<h2 class="wp-block-heading">Changing the energy system to lives and&nbsp;coins</h2>



<p>Currently the game features an energy/fuel bar that slowly drains over time and fills up when collecting the blue pills. I feel like this makes sense but a more conventional system of having lives as health, symbolized by hearts, and coins as goodies is clearer to players and will allow for more flexibility in the gameplay.</p>



<div class="wp-block-image"><figure class="aligncenter is-resized"><img decoding="async" src="https://cdn-images-1.medium.com/max/1600/1*DiGudgI1uLeOeW6YSyBafg.png" alt="" width="677" height="181"/></figure></div>



<div class="wp-block-image"><figure class="aligncenter is-resized"><img loading="lazy" decoding="async" src="https://cdn-images-1.medium.com/max/1600/1*5U9Of77W63w_aTXfN2TGuw.png" alt="" width="622" height="185"/></figure></div>



<p>In the code the change from blue pills to golden coins is easy: We changed the color and then the geometry from <code>THREE.TetrahedronGeometry(5,0)</code> to <code>THREE.CylinderGeometry(4, 4, 1, 10)</code>.</p>



<p>The new logic now is: We start out with three lives and whenever our airplane crashes into an enemy we lose one. The amount of collected coins show in the interface. The coins don’t yet have real impact on the gameplay but they are great for the score board and we can easily add some mechanics later: For example that the player can buy accessoires for the airplane with their coins, having a lifetime coin counter or we could design a game mode where the task is to not miss a single coin on the map.</p>



<h2 class="wp-block-heading">Adding sounds</h2>



<p>This is an obvious improvement and conceptually simple — we just need to find fitting, free sound bites and integrate them.</p>



<p>Luckily on <a href="https://freesound.org" rel="noreferrer noopener" target="_blank">https://freesound.org</a> and <a href="https://www.zapsplat.com/" rel="noreferrer noopener" target="_blank">https://www.zapsplat.com/</a> we can search for sound effects and use them freely, just make sure to attribute where required.</p>



<p>Example of a gun shot sound: <a href="https://freesound.org/people/LeMudCrab/sounds/163456/" rel="noreferrer noopener" target="_blank">https://freesound.org/people/LeMudCrab/sounds/163456/</a>.</p>



<p>We load all 24 sound files at the start of the game and then to play a sound we code a simple <code>audioManager.play(‘shot-soft’)</code>. Repetitively playing the same sound can get boring for the ears when shooting for a few seconds or when collecting a few coins in a row, so we make sure to have several different sounds for those and just select randomly which one to play.</p>



<p>Be aware though that browsers require a page interaction, so basically a mouse click, before they allow a website to play sound. This is to prevent websites from annoyingly auto-playing sounds directly after loading. We can simply require a click on a “Start” button after page load to work around this.</p>



<h2 class="wp-block-heading">Adding collectibles</h2>



<p>How do we get the weapons or new lives to the player? We will spawn “collectibles” for that, which is the item (a heart or gun) floating in a bubble that the player can catch.</p>



<p>We already have the spawning logic in the game, for coins and enemies, so we can adopt that easily.</p>



<pre class="wp-block-code has-small-font-size"><code>class Collectible {
	constructor(mesh, onApply) {
		this.mesh = new THREE.Object3D()
		const bubble = new THREE.Mesh(
			new THREE.SphereGeometry(10, 100, 100),
			new THREE.MeshPhongMaterial({
				color: COLOR_COLLECTIBLE_BUBBLE,
				transparent: true,
				opacity: .4,
				flatShading: true,
			})
		)
		this.mesh.add(bubble)
		this.mesh.add(mesh)
		...
	}


	tick(deltaTime) {
		rotateAroundSea(this, deltaTime, world.collectiblesSpeed)

		// rotate collectible for visual effect
		this.mesh.rotation.y += deltaTime * 0.002 * Math.random()
		this.mesh.rotation.z += deltaTime * 0.002 * Math.random()

		// collision?
		if (utils.collide(airplane.mesh, this.mesh, world.collectibleDistanceTolerance)) {
			this.onApply()
			this.explode()
		}
		// passed-by?
		else if (this.angle &gt; Math.PI) {
			sceneManager.remove(this)
		}
	}


	explode() {
		spawnParticles(this.mesh.position.clone(), 15, COLOR_COLLECTIBLE_BUBBLE, 3)
		sceneManager.remove(this)
		audioManager.play(&#039;bubble&#039;)

		// animation to make it very obvious that we collected this item
		TweenMax.to(...)
	}
}


function spawnSimpleGunCollectible() {
	const gun = SimpleGun.createMesh()
	gun.scale.set(0.25, 0.25, 0.25)
	gun.position.x = -2

	new Collectible(gun, () =&gt; {
		airplane.equipWeapon(new SimpleGun())
	})
}</code></pre>



<p>And that’s it, we have our collectibles:</p>



<div class="wp-block-image"><figure class="aligncenter"><img decoding="async" src="https://cdn-images-1.medium.com/max/1600/1*Qlcdqd92FVR73rTvwegUIA.gif" alt=""/></figure></div>



<p>The only problem is that I couldn’t for the life of me create a heart model from the three.js primitives so I resorted to a <a href="https://www.cgtrader.com/free-3d-models/various/various-models/simple-low-poly-heart" rel="noreferrer noopener" target="_blank">free, low-poly 3D model</a> from the platform cgtrader.</p>



<p>Defining the spawn-logic on the map in a way to have a good balance of challenge and reward requires sensible refining so after some experimenting this felt nice: Spawn the three weapons after a distance of 550, 1150 and 1750 respectively and spawn a life a short while after losing one.</p>



<h2 class="wp-block-heading">Some more&nbsp;polish</h2>



<ul class="wp-block-list"><li>The ocean’s color gets darker as we progress through the levels</li><li>Show more prominently when we enter a new level</li><li>Show an end game screen after 5 levels</li><li>Adjusted the code for a newer version of the Three.js library</li><li>Tweaked the color theme</li></ul>



<h2 class="wp-block-heading">More, more, more&nbsp;fun!</h2>



<p>We went from a simple fly-up-and-down gameplay to being able to collect guns and shoot the enemies. The sounds add to the atmosphere and the coins mechanics sets us up for new features later on.</p>



<p>Make sure to <a href="https://theaviatorgame.vercel.app/" rel="noreferrer noopener" target="_blank"><strong>play our result here</strong></a>! Collect the weapons, have fun with the guns and try to survive until the end of level 5.</p>


