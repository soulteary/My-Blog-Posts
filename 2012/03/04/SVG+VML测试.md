# SVG+VML测试

[![2011061813473885](https://attachment.soulteary.com/2012/03/04/2011061813473885.png "2011061813473885")](https://attachment.soulteary.com/2012/03/04/2011061813473885.png) 

[原文出处.](http://promiseforever.com/redirect?url=http%3A%2F%2Fwww.cnblogs.com%2Fhongru%2Farchive%2F2011%2F06%2F18%2F2084215.html&key=e8a3ae47e9dafeebf0706b543f9351f2) 

半年前看过国外一个大牛的一个demo，然后自己用他的思路写了一个类似的东西。加了点额外的功能。

当时主要是为了学习svg的部分api，做着玩。

<!-- more -->

```
<!DOCTYPE html> 
<html> 
<head> 
<meta charset="utf-8" /> 
<title>Rag Doll</title> 
<meta name="Author" content="hongru.chen" /> 
 <style  _mce_bogus="1"><!--
 
	html {
		overflow: hidden;
	}
	ul,ol {
		margin: 0;
		padding: 0;
		list-style:none;
	}
	body {
		margin: 0px;
		padding: 0px;
		background: #222;
		position: absolute;
		width: 100%;
		height: 100%;
	}
	#screen {
		position: absolute;
		left: 6%;
		top: 10%;
		width: 88%;
		height: 80%;
		background: #000;
		overflow: hidden;
		cursor: default;
	}
	.toolbar {
		position: absolute;
		padding: 20px;
		background: #282828;
		color: #707070;
		right: 20px;
		top: 40px;
		-webkit-border-radius: 10px;
		-moz-border-radius: 10px;
		border-radius: 10px;
	}
	.toolbar label {
		display: inline-block;
		min-width: 60px;
	}
	.toolbar b {
		display: inline-block;
		font-size: 12px;
		color: #b9a662;
		min-width: 30px;
		text-align: center;
	}
	#title {
		position: absolute;
		font-family: verdana;
		width: 100%;
		font-size: 7em;
		font-weight: bold;
		color: #181818;
		text-align: center;
		z-index:0;
	}
	.bar-btn {
		display:inline-block;
		background: #707070;
		color: #b9a662;
		font-size: 10px;
		font-weight: bold;
		text-decoration:none;
		margin-left: 8px;
		padding: 2px 4px;
		-webkit-border-radius: 2px;
		-moz-border-radius: 2px;
		border-radius: 2px;
	}
	.bar-btn.b {font-size: 12px}
	.bar-btn:hover {
		color: #ff3a00;
	}
 
--></style> 
 
<script type="text/javascript"><!--
 
/* ========== for svg and vml compatible ========== */
var __SVG = false;
var __svgNS = false;
if (document.createElementNS) {
	__svgNS = "http://www.w3.org/2000/svg";
	__svg = document.createElementNS(__svgNS, "svg");
	__SVG = (__svg.x != null);
}
if (__SVG) {
	/* ============= SVG ============== */
	vectorGraphics = function(o, antialias) {
		this.canvas = document.createElementNS(__svgNS, "svg");
		this.canvas.style.position = "absolute";
		o.appendChild(this.canvas);
		this.createLine = function(w, col, linecap) {
			var o = document.createElementNS(__svgNS, "line");
			o.setAttribute("shape-rendering", antialias?"auto":"optimizeSpeed");
			o.setAttribute("stroke-width", Math.round(w)+"px");
			if (col) o.setAttribute("stroke", col);
			if (linecap) o.setAttribute("stroke-linecap", linecap);
			o.move = function(x1, y1, x2, y2) {
				this.setAttribute("x1", Math.round(x1) + .5);
				this.setAttribute("y1", Math.round(y1));
				this.setAttribute("x2", Math.round(x2));
				this.setAttribute("y2", Math.round(y2));
			}
			o.color = function(c){ this.setAttribute("stroke", c); }
			o.RGBcolor = function(R, G, B){ this.setAttribute("stroke", "rgb("+Math.round(R)+","+Math.round(G)+","+Math.round(B)+")"); }
			o.stroke_weight = function(s){ this.setAttribute("stroke-width", Math.round(s)+"px"); }
			this.canvas.appendChild(o);
			return o;
		}
		this.createPolyline = function(w, points, col, fill) {
			var o = document.createElementNS(__svgNS, "polyline");
			o.setAttribute("shape-rendering", antialias?"auto":"optimizeSpeed");
			o.setAttribute("stroke-width", Math.round(w));
			if (col) o.setAttribute("stroke", col);
			o.setAttribute("fill", fill?fill:"none");
			if (points) o.setAttribute("points", points);
			o.move = function(points) {
				this.setAttribute("points", points);
			}
			this.canvas.appendChild(o);
			return o;
		}
		this.createOval = function(diam, filled) {
			var o = document.createElementNS(__svgNS, "circle");
			o.setAttribute("shape-rendering", antialias?"auto":"optimizeSpeed");
			o.setAttribute("stroke-width", 0);
			o.setAttribute("r", Math.round(diam / 2));
			o.style.cursor = "pointer";
			o.move = function(x1, y1, radius) {
				this.setAttribute("cx", Math.round(x1));
				this.setAttribute("cy", Math.round(y1));
				this.setAttribute("r", Math.round(radius));
			}
			o.stroke_color = function(col) { this.setAttribute("stroke", col); }
			o.fill_color = function(col) { this.setAttribute("fill", col); }
			o.stroke_weight = function(sw) { this.setAttribute("stroke-width", sw); }
			this.canvas.appendChild(o);
			return o;
		}
	}
	
} else if (document.createStyleSheet) {
	/* ============= VML ============== */
	vectorGraphics = function(o, antialias) {
		document.namespaces.add("v", "urn:schemas-microsoft-com:vml");
		var style = document.createStyleSheet();
		var VMLel = ['line','stroke','polyline','fill','oval'];
		for (var i=0,l=VMLel.length;i<l;i++) {
			style.addRule('v\\:'+VMLel[i], "behavior: url(#default#VML);");
			style.addRule('v\\:'+VMLel[i], "antialias: "+antialias+";");
		}
		this.canvas = o;
		this.createLine = function(w, col, linecap) {
			var o = document.createElement("v:line");
			o.strokeweight = Math.round(w)+"px";
			if (col) o.strokecolor = col;
			o.move = function(x1, y1, x2, y2) {
				this.to   = (Math.round(x1) + .5) + "," + Math.round(y1);
				this.from = Math.round(x2) + "," + Math.round(y2);
			}
			o.color = function(c){ this.strokecolor = c; }
			o.RGBcolor = function(R, G, B){ this.strokecolor = "rgb("+Math.round(R)+","+Math.round(G)+","+Math.round(B)+")"; }
			o.stroke_weight = function(s){ this.strokeweight = Math.round(s)+"px"; }
			if (linecap) {
				s = document.createElement("v:stroke");
				s.endcap = linecap;
				o.appendChild(s);
			}
			this.canvas.appendChild(o);
			return o;
		}
		this.createPolyline = function(w, points, col, fill) {
			var o = document.createElement("v:polyline");
			o.strokeweight = Math.round(w)+"px";
			if (col) o.strokecolor = col;
			o.points = points;
			if (fill) o.fillcolor = fill;
			else {
				s = document.createElement("v:fill");
				s.on = "false";
				o.appendChild(s);
			}
			o.move = function(points) {
				this.points.value = points;
			}
			this.canvas.appendChild(o);
			return o;
		}
		this.createOval = function(diam, filled) {
			var o = document.createElement("v:oval");
			var os = o.style;
			os.position = "absolute";
			os.cursor = "pointer";
			o.strokeweight = 1;
			o.filled = filled;
			os.width = Math.round(diam) + "px";
			os.height = Math.round(diam) + "px";
			o.move = function(x1, y1, radius) {
				os.left   = Math.round(x1 - radius) + "px";
				os.top    = Math.round(y1 - radius) + "px";
				os.width  = Math.round(radius * 2) + "px";
				os.height = Math.round(radius * 2) + "px";
			}
			o.stroke_color = function(col) { this.strokecolor = col; }
			o.fill_color = function(col) { this.fillcolor = col; }
			o.stroke_weight = function(sw) { this.strokeweight = sw; }
			this.canvas.appendChild(o);
			return o;
		}
	}
} else {
	/* ==== no script ==== */
	vectorGraphics = function(o, i) {
		return false;
	}
}
 
/* ====== Vector ====== */
var Vector = function (x, y) {
	this.x = x;
	this.y = y;
};
Vector.prototype.equal = function (v, x, y) {
	this.x = v.x || x || 0;
	this.y = v.y || y || 0;
};
Vector.prototype.add = function (v, x, y) {
	this.x += v.x || x || 0;
	this.y += v.y || y || 0;
}
 
 
/* ====== main ========*/
var RagDoll = function () {
	//private variables
	var M = {},
		S = {},
		D = {},
		dolls = [],
		nodes = [],
		blood = [],
		collisions = [],
		svg,
		scr,
		zoom,
		dragDoll = false;
		nBlood = 100;
	
	//private methods
	var _ = {
		$: function (id) {return document.getElementById(id)},
		addEvent: function (o, e, f) {
			o.addEventListener ? o.addEventListener(e, f, false) : o.attachEvent('on'+e, function () {f.call(o)})
		},
		extend: function (t, s) {
			for (var p in s) {
				t[p] = s[p]
			}
			return t;
		},
		getPos: function (el) {
			for (var pos = {x:0, y:0}; el; el = el.offsetParent) {
				pos.x += el.offsetLeft;
				pos.y += el.offsetTop;
			}
			return pos;
		},
		getStyle : function (p, el) {
            return el.currentStyle ? el.currentStyle[p] : document.defaultView.getComputedStyle(el, null).getPropertyValue(p);
        },
		randomInt: function (f, t) {
			return Math.ceil(f - 1 + Math.random()*(t-f));
		},
		randomColor: function () {
			var cc = '0123456789abcdef'.split(''), color = '#';
			for (var i = 0; i < 6; i ++) {
				color += cc[_.randomInt(0, 16)];
			}
			return color;
		}
	}
	
	function initScript () {
		for (var i=0; i<nBlood; i++) {blood.push(new Blood())}
		for (var i=0; i<dolls.length; i++) {
			for (var j=0; j<i; j++) {
				if (i != j) {
				// for collosions between dolls
					var o1 = dolls[i],
						o2 = dolls[j];
					for (var i1 = 0, n1; n1 = o1.nodes[i1++]; ) {
						for (var i2 = 0, n2; n2 = o2.nodes[i2++]; ) {
							if (n1.testCol == 1 && n2.testCol == 1) {
								collisions.push({
									n1p: n1.pos,
									n1o: n1.oBody.pos,
									n2p: n2.pos,
									n2o: n2.oBody.pos,
									n2s: new Vector(0, 0),
									r   : ((n1.size * .5) + (n2.size * .5)) / n1.h2Body,
									m2  : n1.mass / (n1.mass + n2.mass),
									m1  : n2.mass / (n1.mass + n2.mass)
								})
							}
						}
					}
				}
			}
		}
	}
	
	// Tool Bar
	function initToolBar () {
		var bar = document.createElement('div');
		bar.className = 'toolbar';
		bar.innerHTML = '<ul><li><label>bloody:</label><input class="ck" type="checkbox" checked="true" id="bloody-ck" /></li></li><li><label>gravity:</label><b id="gravity-text"></b><a id="gravity-up" class="bar-btn" href="javascript:;">+</a><a id="gravity-down" class="bar-btn" href="javascript:;">-</a></li><li><label>viscosity:</label><b id="viscosity-text"></b><a id="viscosity-up" class="bar-btn" href="javascript:;">+</a><a id="viscosity-down" class="bar-btn" href="javascript:;">-</a></li><li><label>flexibility:</label><b id="flexibility-text"></b><a id="flexibility-up" class="bar-btn" href="javascript:;">+</a><a id="flexibility-down" class="bar-btn" href="javascript:;">-</a></li><li><label>collision:</label><b id="collision-text"></b><a id="collision-up" class="bar-btn" href="javascript:;">+</a><a id="collision-down" class="bar-btn" href="javascript:;">-</a></li><li><a id="shake" class="bar-btn b" href="javascript:;">shake</a></li></ul>';
		scr.appendChild(bar);
		
		initBarEvents();
	}
	
	function pushDolls (x, y, n) {
		//[node_ind,node_parent_ind,node_length,node_spring_parent_ind,spring_length,node_width,node_mass,node_color,blood_on_off,collisions_on_off]
		var c0 = _.randomColor(), c1 = _.randomColor(), c2 = _.randomColor(), c3 = _.randomColor();
		var dolllist = [
			// person
			[[1,0,3,7,80,10,1, c0, 0, 0],
			[5,3,20,1,60,8,1, c0, 0, 1],
			[6,4,20,1,60,8,1, c0, 0, 1],
			[3,1,20,4,40,10,1, c1, 1, 1],
			[4,1,20,2,40,10,1, c1, 1, 1],
			[9,7,30,2,80,8,2, c0, 0, 1],
			[10,8,30,2,80,8,2, c0, 0, 1],
			[7,2,30,8,30,13,1, c2, 1, 1],
			[8,2,30,1,80,13,1, c2, 1, 1],
			[2,1,30,3,40,20,2, c1, 1, 1],
			[0,0,35,2,60,35,1, c0, 1, 1],
			[11,1,1,2,25,0,1, c3, 0, 0],
			[12,11,25,11,25,5,1, c0, 0, 0]],
			// rect
			[[1,2,100*.5,3,141*.5,20,2, c0, 0, 1], 
			[2,3,100*.5,0,141*.5,20,2, c0, 0, 1], 
			[3,0,100*.5,1,141*.5,20,2, c0, 0, 1], 
			[0,1,100*.5,2,141*.5,20,2, c0, 0, 1]],
			// triangle
			[[1,2,100*.5,0,141*.5,20,4, c1, 0, 1], 
			[2,0,100*.5,1,141*.5,20,4, c1, 0, 1], 
			[0,1,100*.5,2,141*.5,20,2, c1, 0, 1]],
			// point
			[[0,0,10,0,10,_.randomInt(10, 40),.1, _.randomColor(), 0, 2]]
		]
		
		if (n === undefined) {
			zoom = S.h/500;
			dolls.push(new Doll(x, y, [
				[1,0,3,7,80,10,1, "#fff", 0, 0],
				[5,3,20,1,60,8,1, "#fff", 0, 1],
				[6,4,20,1,60,8,1, "#fff", 0, 1],
				[3,1,20,4,40,10,1, "#FF7a00", 1, 1],
				[4,1,20,2,40,10,1, "#FF7a00", 1, 1],
				[9,7,30,2,80,8,2, "#fff", 0, 1],
				[10,8,30,2,80,8,2, "#fff", 0, 1],
				[7,2,30,8,30,13,1, "#333", 1, 1],
				[8,2,30,1,80,13,1, "#333", 1, 1],
				[2,1,30,3,40,20,2, "#FF7a00", 1, 1],
				[0,0,35,2,60,35,1, "#fff", 1, 1],
				[11,1,1,2,25,0,1, "#da4901", 0, 0],
				[12,11,25,11,25,5,1, "#fff", 0, 0]
			]))
		} else {
			while (n > 0) {
				zoom = Math.random();
				var _i = _.randomInt(0, 4),
					_x = x == 'random' ? Math.random() * S.w : x;
					_y = y == 'random' ? Math.random() * S.h : y;
				dolls.push(new Doll(_x, _y, dolllist[_i]));		
				n --;
			}
		}
		
	}
	
	var sking = false;
	function shakeBox (cb) {
		if (!sking) {
			var ol = parseInt(_.getStyle('left', scr), 10), ot = parseInt(_.getStyle('top', scr), 10), dl = 20, dt = 20, f = -1;
			(function () {
				if (dl > 0) {
					sking = true;
					scr.style['left'] = ol + f*dl + 'px';
					f *= -1;
					dl -= 4;
				} else if (dt > 0) {	
					sking = true;
					scr.style['top'] = ot + f*dt + 'px';
					f *= -1;
					dt -= 4;
				} else {
					sking = false;
					scr.style['left'] = ol + 'px';
					scr.style['top'] = ot + 'px';
					!!cb && cb();
					return;
				}
				setTimeout(arguments.callee, 100);
			})()
		}
	}
	
	function initBarEvents () {
		var g = {
			u: _.$('gravity-up'),
			d: _.$('gravity-down'),
			t: _.$('gravity-text')
		},
		v = {
			u: _.$('viscosity-up'),
			d: _.$('viscosity-down'),
			t: _.$('viscosity-text')
		},
		f = {
			u: _.$('flexibility-up'),
			d: _.$('flexibility-down'),
			t: _.$('flexibility-text')
		},
		c = {
			u: _.$('collision-up'),
			d: _.$('collision-down'),
			t: _.$('collision-text')
		},
		ck = _.$('bloody-ck'),
		sk = _.$('shake');
		
		_.addEvent(ck, 'click', function () { RagDoll.eBlood = ck.checked })
		_.addEvent(g.u, 'click', function () { RagDoll.gravity += .1; g.t.innerHTML = Math.round(RagDoll.gravity*100)/100 })
		_.addEvent(g.d, 'click', function () { RagDoll.gravity -= .1; g.t.innerHTML = Math.round(RagDoll.gravity*100)/100 })
		_.addEvent(v.u, 'click', function () {
			if (RagDoll.viscosity >= 1.1) {alert('viscosity can not be more than '+RagDoll.viscosity+''); return}
			RagDoll.viscosity += .02; v.t.innerHTML = Math.round(RagDoll.viscosity*100)/100 })
		_.addEvent(v.d, 'click', function () { RagDoll.viscosity -= .02; v.t.innerHTML = Math.round(RagDoll.viscosity*100)/100 })
		_.addEvent(f.u, 'click', function () { RagDoll.flexibility += .1; f.t.innerHTML = Math.round(RagDoll.flexibility*100)/100})
		_.addEvent(f.d, 'click', function () { 
			if (Math.floor(RagDoll.flexibility*10) <= 4) {alert('flexibility can not be lesser than 0.4'); return}
			RagDoll.flexibility -= .1; f.t.innerHTML = Math.round(RagDoll.flexibility*100)/100})
		_.addEvent(c.u, 'click', function () { RagDoll.collision += .05; c.t.innerHTML = Math.round(RagDoll.collision*100)/100 })
		_.addEvent(c.d, 'click', function () { RagDoll.collision -= .05; c.t.innerHTML = Math.round(RagDoll.collision*100)/100 })
		_.addEvent(sk, 'click', function () {shakeBox(function () {pushDolls('random', 20, _.randomInt(3, 6))})});
		
		g.t.innerHTML = RagDoll.gravity;
		v.t.innerHTML = RagDoll.viscosity;
		f.t.innerHTML = RagDoll.flexibility;
		c.t.innerHTML = RagDoll.collision;
	}
	function initEvents () {
		// mousemove event
		_.addEvent(document, 'mousemove', function (e) {
			e = e || window.event;
			M.x = e.clientX;
			M.y = e.clientY;
		})
		// mouseup event
		_.addEvent(document, 'mouseup', function (e) {
			if (dragDoll && document.releaseCapture) {
				dragDoll.dragNode.o.releaseCapture();
			}
			scr.style.cursor = 'default';
			dragDoll = false;
		})
		
		resize();
		_.addEvent(window, 'resize', resize);
	}
	
	function resize () {
		S.w = scr.offsetWidth || 0;
		S.h = scr.offsetHeight || 0;
		scr.style['left'] = document.body.offsetWidth * .06 + 'px';
		scr.style['top'] = document.body.offsetHeight * .1 + 'px';
		//S.l = _.getPos(scr).x;
		//S.t = _.getPos(scr).y;
	}
	
	// -- Blood Constructor
	var Blood = function () {
		this.o = svg.createLine(5*zoom, '#f00', 'round');
		this.o.move(-99, -99, -99, -99);
		this.pos = new Vector(0, 0);
		this.vel = new Vector(0, 0);
		this.bloody = false;
	}
	_.extend(Blood.prototype, {
		start: function (x, y, vx, vy) {
			this.pos.equal(false, x, y);
			this.vel.equal(false, vx, vy);
			this.bloody = true;
		},
		anim: function () {
			this.vel.x *= .9;
			this.vel.y *= .9;
			this.pos.add(this.vel);
			
			if (Math.abs(this.vel.x) + Math.abs(this.vel.y) > 1) {
				this.o.move(this.pos.x, this.pos.y, this.pos.x+this.vel.x, this.pos.y+this.vel.y)
			} else {
				this.o.move(-99, -99, -99, -99);
				this.bloody = false;
			}
		}
	})
	
	// --- Doll Constructor
	var Doll = function (x, y, params) {
		this.nodes = [];
		
		this.Node = function (doll, p) {
			this.parent = doll;
			this.ind = p[0];
			this.oBody  = p[1];
			this.oSpring = p[3];
			this.hBody = p[2]*p[2]*zoom*zoom;
			this.h2Body = p[2]*zoom;
			this.hSpring = p[4]*p[4]*zoom*zoom;
			this.size = p[5]*zoom;
			this.mass = p[6];
			this.bloody = p[8];
			this.testCol = p[9];
			this.pos = new Vector(x + Math.random(), y);
			this.old = new Vector(x + Math.random(), y);
			this.vel = new Vector(0, 0);
			
			this.o = svg.createLine(this.size, p[7], 'round');
			this.o.style.cursor = 'pointer';
			this.o.parent = this;
			this.s = false;
			
			// drag events
			this.o.onselectstart = function () { return false }
			this.o.ondrag = function () { return false }
			this.o.onmousedown = function () {
				dragDoll = this.parent.parent;
				dragDoll.dragNode = this.parent;
				D.x = dragDoll.dragNode.pos.x - M.x;
				D.y = dragDoll.dragNode.pos.y - M.y;
				scr.style.cursor = 'pointer';
				this.setCapture && this.setCapture();
				return false;
			}
			//console.log(doll)
			doll.nodes[this.ind] = this;
			nodes.push(this);
		}
		_.extend(this.Node.prototype, {
			// blood splash
			bounce: function (val) {
				if (this.bloody) {
					if (this.vel.x*this.vel.x + this.vel.y*this.vel.y > zoom) {
						for (var i=2; i<2+Math.random(); i+=.25) {
							if (RagDoll.eBlood) {
								blood[Math.floor(Math.random()*nBlood)].start(this.pos.x, this.pos.y, .001-this.vel.x*zoom*i, .001-this.vel.y*zoom*i);
							}
						}
					}
				}
				this.old.equal(this.pos);
				return val;
			},
			verlet: function () {
				this.vel.x = (RagDoll.viscosity * (this.pos.x - this.old.x)) || 1;
				this.vel.y = (RagDoll.viscosity * (this.pos.y - this.old.y) + RagDoll.gravity) || 1;
				this.old.equal(this.pos);
				this.pos.add(this.vel);
				
				if (this.testCol) {
					var size = this.size * .5;
					if (this.pos.x < size) this.pos.x = this.bounce(size);
					else if (this.pos.x > S.w - size) this.pos.x = this.bounce(S.w - size);
					if (this.pos.y < size) this.pos.y = this.bounce(size);
					else if (this.pos.y > S.h - size) this.pos.y = this.bounce(S.h - size);
				}
				
				if (this != this.oSpring) this.satisfyConstraints(this.oSpring, this.hSpring);
				if (this != this.oBody) this.satisfyConstraints(this.oBody, this.hBody);
			},
			satisfyConstraints: function (that, len) {
				var dx = that.pos.x - this.pos.x;
				var dy = that.pos.y - this.pos.y;
				var delta = len / (dx * dx + dy * dy + len) - .5;
				var m1 = (this.mass + that.mass) * RagDoll.flexibility;
				var m2 = this.mass / m1;
				m1 = that.mass / m1;
				this.pos.add(false, -m1 * dx * delta, -m1 * dy * delta);
				that.pos.add(false,  m2 * dx * delta,  m2 * dy * delta);
			},
			rendering: function () {
				/* ---- body ---- */
				if (this.size) this.o.move(
					this.pos.x, this.pos.y,
					this.oBody.pos.x, this.oBody.pos.y
				);
				/* ---- springs ---- */
				if (this.s)
					this.s.move(
						this.pos.x, this.pos.y,
						this.oSpring.pos.x, this.oSpring.pos.y
					);
			}
		})
		
		/* == create Nodes == */
		for (var i = 0, p; p = params[i++];) { new this.Node(this, p) }
		for (var i = 0, o; o = this.nodes[i++];) {
			o.oBody = this.nodes[o.oBody];
			o.oSpring = this.nodes[o.oSpring];
		}
	}
	
	// segments intersection
	function segmentsIntersection (p1, p2, p3, p4) {
		var dn = ((p4.y - p3.y) * (p2.x - p1.x)) - ((p4.x - p3.x) * (p2.y - p1.y));
		if (dn) {
			var na = ((p4.x - p3.x) * (p1.y - p3.y)) - ((p4.y - p3.y) * (p1.x - p3.x));
			var nb = ((p2.x - p1.x) * (p1.y - p3.y)) - ((p2.y - p1.y) * (p1.x - p3.x));
			var ua = na / dn;
			var ub = nb / dn;
			if (ua >= 0 && ua <= 1 && ub >= 0 && ub <= 1) return ua;
		}
		return false;
	}
	
	// ==== Main Loop
	var run = function () {
		if (dragDoll) {
			dragDoll.dragNode.pos.x += (M.x + D.x - dragDoll.dragNode.pos.x) * .2;
			dragDoll.dragNode.pos.y += (M.y + D.y - dragDoll.dragNode.pos.y) * .2;
		}
		// draw doll
		for (var i = 0, o; o = nodes[i++]; ) o.verlet();
		for (var i = 0, o; o = nodes[i++]; ) o.rendering();
		// draw blood
		for (var i = 0, o; o = blood[i++]; ) o.anim();
		// collision between dolls
		if (RagDoll.collision) {
			for (var i=0, o; o = collisions[i++]; ) {
				o.n2s.x = o.n2p.x - (o.n2o.x - o.n2p.x) * o.r;
				o.n2s.y = o.n2p.y - (o.n2o.y - o.n2p.y) * o.r;
				var ua = segmentsIntersection(o.n1p, o.n1o, o.n2s, o.n2o);
				if (ua != false) {			
					var ix = ua * (o.n2o.x - o.n2p.x) * RagDoll.collision;
					var iy = ua * (o.n2o.y - o.n2p.y) * RagDoll.collision;
					o.n1o.x -= ix * o.m1;
					o.n2o.x += ix * o.m2;
					o.n1o.y -= iy * o.m1;
					o.n2o.y += iy * o.m2;
				}
			}
		}
		
		setTimeout(run, 16);
	}
	
	return {
		eBlood: true,
		viscosity   : 1,
		flexibility : 1,
		gravity     : .1,
		collision   : .05,
		
		init: function () {
			_.addEvent(window, 'load', function () {
				scr = _.$('screen');
				initEvents();
				svg = new vectorGraphics(scr, true);
				pushDolls(S.w*.5, S.h*.5)
				initScript();
				initToolBar();
				run();
			})
		}
	}
}();
 
RagDoll.init();
// --></script> 
 
</head> 
 
<body> 
	<div id="screen"> 
		<div id="title">Rag Doll</div>               
	</div> 
</body> 
</html> 

</runcode>

里面有做svg和vml的兼容，所以ie6.0+也ok的，但是效率还是很慢。（画直线，折线，和椭圆）
右上角控制栏里的 “shake”和各种参数都可以点下试试看。

<pre lang="javascript">
/* ========== for svg and vml compatible ========== */
var __SVG = false;
var __svgNS = false;
if (document.createElementNS) {
    __svgNS = "http://www.w3.org/2000/svg";
    __svg = document.createElementNS(__svgNS, "svg");
    __SVG = (__svg.x != null);
}
if (__SVG) {
    /* ============= SVG ============== */
    vectorGraphics = function(o, antialias) {
        this.canvas = document.createElementNS(__svgNS, "svg");
        this.canvas.style.position = "absolute";
        o.appendChild(this.canvas);
        this.createLine = function(w, col, linecap) {
            var o = document.createElementNS(__svgNS, "line");
            o.setAttribute("shape-rendering", antialias?"auto":"optimizeSpeed");
            o.setAttribute("stroke-width", Math.round(w)+"px");
            if (col) o.setAttribute("stroke", col);
            if (linecap) o.setAttribute("stroke-linecap", linecap);
            o.move = function(x1, y1, x2, y2) {
                this.setAttribute("x1", Math.round(x1) + .5);
                this.setAttribute("y1", Math.round(y1));
                this.setAttribute("x2", Math.round(x2));
                this.setAttribute("y2", Math.round(y2));
            }
            o.color = function(c){ this.setAttribute("stroke", c); }
            o.RGBcolor = function(R, G, B){ this.setAttribute("stroke", "rgb("+Math.round(R)+","+Math.round(G)+","+Math.round(B)+")"); }
            o.stroke_weight = function(s){ this.setAttribute("stroke-width", Math.round(s)+"px"); }
            this.canvas.appendChild(o);
            return o;
        }
        this.createPolyline = function(w, points, col, fill) {
            var o = document.createElementNS(__svgNS, "polyline");
            o.setAttribute("shape-rendering", antialias?"auto":"optimizeSpeed");
            o.setAttribute("stroke-width", Math.round(w));
            if (col) o.setAttribute("stroke", col);
            o.setAttribute("fill", fill?fill:"none");
            if (points) o.setAttribute("points", points);
            o.move = function(points) {
                this.setAttribute("points", points);
            }
            this.canvas.appendChild(o);
            return o;
        }
        this.createOval = function(diam, filled) {
            var o = document.createElementNS(__svgNS, "circle");
            o.setAttribute("shape-rendering", antialias?"auto":"optimizeSpeed");
            o.setAttribute("stroke-width", 0);
            o.setAttribute("r", Math.round(diam / 2));
            o.style.cursor = "pointer";
            o.move = function(x1, y1, radius) {
                this.setAttribute("cx", Math.round(x1));
                this.setAttribute("cy", Math.round(y1));
                this.setAttribute("r", Math.round(radius));
            }
            o.stroke_color = function(col) { this.setAttribute("stroke", col); }
            o.fill_color = function(col) { this.setAttribute("fill", col); }
            o.stroke_weight = function(sw) { this.setAttribute("stroke-width", sw); }
            this.canvas.appendChild(o);
            return o;
        }
    }
     
} else if (document.createStyleSheet) {
    /* ============= VML ============== */
    vectorGraphics = function(o, antialias) {
        document.namespaces.add("v", "urn:schemas-microsoft-com:vml");
        var style = document.createStyleSheet();
        var VMLel = ['line','stroke','polyline','fill','oval'];
        for (var i=0,l=VMLel.length;i<l;i++) {
            style.addRule('v\\:'+VMLel[i], "behavior: url(#default#VML);");
            style.addRule('v\\:'+VMLel[i], "antialias: "+antialias+";");
        }
        this.canvas = o;
        this.createLine = function(w, col, linecap) {
            var o = document.createElement("v:line");
            o.strokeweight = Math.round(w)+"px";
            if (col) o.strokecolor = col;
            o.move = function(x1, y1, x2, y2) {
                this.to   = (Math.round(x1) + .5) + "," + Math.round(y1);
                this.from = Math.round(x2) + "," + Math.round(y2);
            }
            o.color = function(c){ this.strokecolor = c; }
            o.RGBcolor = function(R, G, B){ this.strokecolor = "rgb("+Math.round(R)+","+Math.round(G)+","+Math.round(B)+")"; }
            o.stroke_weight = function(s){ this.strokeweight = Math.round(s)+"px"; }
            if (linecap) {
                s = document.createElement("v:stroke");
                s.endcap = linecap;
                o.appendChild(s);
            }
            this.canvas.appendChild(o);
            return o;
        }
        this.createPolyline = function(w, points, col, fill) {
            var o = document.createElement("v:polyline");
            o.strokeweight = Math.round(w)+"px";
            if (col) o.strokecolor = col;
            o.points = points;
            if (fill) o.fillcolor = fill;
            else {
                s = document.createElement("v:fill");
                s.on = "false";
                o.appendChild(s);
            }
            o.move = function(points) {
                this.points.value = points;
            }
            this.canvas.appendChild(o);
            return o;
        }
        this.createOval = function(diam, filled) {
            var o = document.createElement("v:oval");
            var os = o.style;
            os.position = "absolute";
            os.cursor = "pointer";
            o.strokeweight = 1;
            o.filled = filled;
            os.width = Math.round(diam) + "px";
            os.height = Math.round(diam) + "px";
            o.move = function(x1, y1, radius) {
                os.left   = Math.round(x1 - radius) + "px";
                os.top    = Math.round(y1 - radius) + "px";
                os.width  = Math.round(radius * 2) + "px";
                os.height = Math.round(radius * 2) + "px";
            }
            o.stroke_color = function(col) { this.strokecolor = col; }
            o.fill_color = function(col) { this.fillcolor = col; }
            o.stroke_weight = function(sw) { this.strokeweight = sw; }
            this.canvas.appendChild(o);
            return o;
        }
    }
} else {
    /* ==== no script ==== */
    vectorGraphics = function(o, i) {
        return false;
    }
}
```


关于画矢量图（直线，折线，圆），下面是个简单的演示，利用上面的代码：

Run Code:

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8" />
<script type="text/javascript"><!--
/* ========== for svg and vml compatible ========== */
var __SVG = false;
var __svgNS = false;
if (document.createElementNS) {
	__svgNS = "http://www.w3.org/2000/svg";
	__svg = document.createElementNS(__svgNS, "svg");
	__SVG = (__svg.x != null);
}
if (__SVG) {
	/* ============= SVG ============== */
	vectorGraphics = function(o, antialias) {
		this.canvas = document.createElementNS(__svgNS, "svg");
		this.canvas.style.position = "absolute";
		o.appendChild(this.canvas);
		this.createLine = function(w, col, linecap) {
			var o = document.createElementNS(__svgNS, "line");
			o.setAttribute("shape-rendering", antialias?"auto":"optimizeSpeed");
			o.setAttribute("stroke-width", Math.round(w)+"px");
			if (col) o.setAttribute("stroke", col);
			if (linecap) o.setAttribute("stroke-linecap", linecap);
			o.move = function(x1, y1, x2, y2) {
				this.setAttribute("x1", Math.round(x1) + .5);
				this.setAttribute("y1", Math.round(y1));
				this.setAttribute("x2", Math.round(x2));
				this.setAttribute("y2", Math.round(y2));
			}
			o.color = function(c){ this.setAttribute("stroke", c); }
			o.RGBcolor = function(R, G, B){ this.setAttribute("stroke", "rgb("+Math.round(R)+","+Math.round(G)+","+Math.round(B)+")"); }
			o.stroke_weight = function(s){ this.setAttribute("stroke-width", Math.round(s)+"px"); }
			this.canvas.appendChild(o);
			return o;
		}
		this.createPolyline = function(w, points, col, fill) {
			var o = document.createElementNS(__svgNS, "polyline");
			o.setAttribute("shape-rendering", antialias?"auto":"optimizeSpeed");
			o.setAttribute("stroke-width", Math.round(w));
			if (col) o.setAttribute("stroke", col);
			o.setAttribute("fill", fill?fill:"none");
			if (points) o.setAttribute("points", points);
			o.move = function(points) {
				this.setAttribute("points", points);
			}
			this.canvas.appendChild(o);
			return o;
		}
		this.createOval = function(diam, filled) {
			var o = document.createElementNS(__svgNS, "circle");
			o.setAttribute("shape-rendering", antialias?"auto":"optimizeSpeed");
			o.setAttribute("stroke-width", 1);
			o.setAttribute("r", Math.round(diam / 2));
            o.setAttribute("fill", filled);
			o.style.cursor = "pointer";
			o.move = function(x1, y1, radius) {
				this.setAttribute("cx", Math.round(x1));
				this.setAttribute("cy", Math.round(y1));
				this.setAttribute("r", Math.round(radius));
			}
			o.stroke_color = function(col) { this.setAttribute("stroke", col); }
			o.fill_color = function(col) { this.setAttribute("fill", col); }
			o.stroke_weight = function(sw) { this.setAttribute("stroke-width", sw); }
			this.canvas.appendChild(o);
			return o;
		}
	}
	
} else if (document.createStyleSheet) {
	/* ============= VML ============== */
	vectorGraphics = function(o, antialias) {
		document.namespaces.add("v", "urn:schemas-microsoft-com:vml");
		var style = document.createStyleSheet();
		var VMLel = ['line','stroke','polyline','fill','oval'];
		for (var i=0,l=VMLel.length;i<l;i++) {
			style.addRule('v\\:'+VMLel[i], "behavior: url(#default#VML);");
			style.addRule('v\\:'+VMLel[i], "antialias: "+antialias+";");
		}
		this.canvas = o;
		this.createLine = function(w, col, linecap) {
			var o = document.createElement("v:line");
			o.strokeweight = Math.round(w)+"px";
			if (col) o.strokecolor = col;
			o.move = function(x1, y1, x2, y2) {
				this.to   = (Math.round(x1) + .5) + "," + Math.round(y1);
				this.from = Math.round(x2) + "," + Math.round(y2);
			}
			o.color = function(c){ this.strokecolor = c; }
			o.RGBcolor = function(R, G, B){ this.strokecolor = "rgb("+Math.round(R)+","+Math.round(G)+","+Math.round(B)+")"; }
			o.stroke_weight = function(s){ this.strokeweight = Math.round(s)+"px"; }
			if (linecap) {
				s = document.createElement("v:stroke");
				s.endcap = linecap;
				o.appendChild(s);
			}
			this.canvas.appendChild(o);
			return o;
		}
		this.createPolyline = function(w, points, col, fill) {
			var o = document.createElement("v:polyline");
			o.strokeweight = Math.round(w)+"px";
			if (col) o.strokecolor = col;
			o.points = points;
			if (fill) o.fillcolor = fill;
			else {
				s = document.createElement("v:fill");
				s.on = "false";
				o.appendChild(s);
			}
			o.move = function(points) {
				this.points.value = points;
			}
			this.canvas.appendChild(o);
			return o;
		}
		this.createOval = function(diam, filled) {
			var o = document.createElement("v:oval");
			var os = o.style;
			os.position = "absolute";
			os.cursor = "pointer";
			o.strokeweight = 0;
			o.filled = filled;
            o.fillcolor = filled;
			os.width = Math.round(diam) + "px";
			os.height = Math.round(diam) + "px";
			o.move = function(x1, y1, radius) {
				os.left   = Math.round(x1 - radius) + "px";
				os.top    = Math.round(y1 - radius) + "px";
				os.width  = Math.round(radius * 2) + "px";
				os.height = Math.round(radius * 2) + "px";
			}
			o.stroke_color = function(col) { this.strokecolor = col; }
			o.fill_color = function(col) { this.fillcolor = col; }
			o.stroke_weight = function(sw) { this.strokeweight = sw; }
			this.canvas.appendChild(o);
			return o;
		}
	}
} else {
	/* ==== no script ==== */
	vectorGraphics = function(o, i) {
		return false;
	}
}
// --></script>
<style  _mce_bogus="1"><!--
#screen {
    height: 400px;
    background: #f3f3f3;
    width: 80%;
    margin: 20px 10%;
    border: 1px solid #d5d5d5;
}
.hide{display:none}
--></style>
</head>
<body>
<div id="screen"></div>
<input id="draw-line" type="button" value="绘制矢量图" />
<input id="rotate-line" class="hide" type="button" value="变换" />
<script type="text/javascript"><!--
var addEvent = function (o, e, f) {
    o.addEventListener ? o.addEventListener(e, f, false) : o.attachEvent('on'+e, function () {f.call(o)});
},
    $ = function (id) { return document.getElementById(id) }
onload = function () {
    var scr = document.getElementById('screen'),
        svg = new vectorGraphics(scr, true),
        
        dl = $('draw-line'),
        rl = $('rotate-line');
        
    
    var Line = function (weight, color, r) {
        this.o = svg.createLine(weight, color, 'round');
        this.pl = svg.createPolyline(2, '', color, '#d8e');
        this.c = svg.createOval(0, color);
        
        this.a = 0;
        this.r = r;
        this.pos = {x: 20, y: 30};
        this.pos2 = {x: 400, y: 100};
    }
    Line.prototype = {
        show: function () {
            this.o.move(this.r*(1+Math.sin(this.a))+this.pos.x, this.r*(1-Math.cos(this.a))+this.pos.y, this.r*(1-Math.sin(this.a))+this.pos.x, this.r*(1+Math.cos(this.a))+this.pos.y);
            this.pl.move(''+(this.pos2.x-this.r*Math.sin(30+this.a))+','+(this.pos2.y-this.r*Math.cos(30+this.a))+' '+(this.pos2.x+this.r*Math.sin(30+this.a))+','+(this.pos2.y-this.r*Math.cos(30+this.a))+' '+(this.pos2.x+this.r*Math.sin(this.a))+','+(this.pos2.y+this.r*Math.cos(this.a))+'');
            
            this.c.move(800, 160, this.r*(Math.cos(this.a)+1.5)/2);
        },
        rotate: function () {
            var _this = this;
            this._interval = setInterval(function () {
                _this.show();
                _this.a += .1;
            }, 16)
        },
        stop: function () {
            clearInterval(this._interval)
        }
    }
    
    var line = new Line(10, '#da4901', 80);
    addEvent(dl, 'click', function () {
        line.show();
        rl.style['display'] = 'inline'
    });
    addEvent(rl, 'click', function () {
        if (this.value == '变换') {
            line.rotate();
            this.value = '停止';
        } else if (this.value == '停止') {
            line.stop();
            this.value = '变换';
        }  
    })
}
// --></script>
</body>
</html>
```

在做浏览器适配的时候，某些情况下不得不考虑ie低版本的兼容时，其实svg+vml在矢量图方面算个不错的选择。

利用上面的直线和圆，其实就可以做个时钟了。: )


