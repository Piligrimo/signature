<template>
  <div id="app">
    <canvas id="canvas" ref="canvas"></canvas>
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      pos: {
        x: 0,
        y: 0
      },
      socket: null,
      lastSent: new Date()
    }
  },
  computed: {
    canvas() {
      return this.$refs.canvas;
    },
    ctx() {
      return this.canvas.getContext('2d');
    }
  },
  methods: { 
    setPosition(e) {
      const {top, left } = this.canvas.getBoundingClientRect()
      this.pos.x = e.clientX - left;
      this.pos.y = e.clientY - top;
    },
    resize() {
      this.ctx.canvas.width = this.canvas.offsetWidth;
      this.ctx.canvas.height = this.canvas.offsetHeight;
    },
    sendImage() {
      if (new Date() - this.lastSent > 300) {
        const url = this.canvas.toDataURL();
        this.lastSent = new Date();
        this.socket.send(JSON.stringify(url)) 
      }    
    },
    sendLine(start, finish) {
      const str = JSON.stringify({start, finish})
      console.log('sendLine', str);

      this.socket.send(str);
    },
    getImage(serverImage) {
      var img=new Image();
      img.onload= () => {
        URL.revokeObjectURL(event.target.src)
        this.ctx.drawImage(serverImage,0,0);
      };
    },
    draw(e) {
      if (e.buttons !== 1) return;
      const start = {...this.pos};

      this.ctx.beginPath(); // begin

      this.ctx.lineWidth = 5;
      this.ctx.lineCap = 'round';
      this.ctx.strokeStyle = '#c0392b';

      this.ctx.moveTo(this.pos.x, this.pos.y); // from
      this.setPosition(e);
      this.ctx.lineTo(this.pos.x, this.pos.y); // to
      const finish = {...this.pos};
      this.ctx.stroke(); // draw it!
      this.sendLine(start, finish);
    },
  },
  mounted() {
    this.socket = new WebSocket('wss://wiry-pastoral-spade.glitch.me');
    this.socket.addEventListener('open', function () {
        console.log('Connected to WS Server')
    });

    this.socket.addEventListener('message', (event) => {
      try {
        const {start, finish} = JSON.parse(event.data);
        this.ctx.lineWidth = 5;
        this.ctx.lineCap = 'round';
        this.ctx.strokeStyle = '#c0392b';
        this.ctx.moveTo(start.x, start.y); // from
        this.ctx.lineTo(finish.x, finish.y); // to
        this.ctx.stroke(); // draw it!
      } catch (error) {
        console.error(error);
      }
    })

    this.resize();
    window.addEventListener('resize', this.resize);
    document.addEventListener('mousemove', this.draw);
    document.addEventListener('mousedown', this.setPosition);
    document.addEventListener('touchmove', this.draw);
    document.addEventListener('touchstart', this.setPosition);
    document.addEventListener('mouseenter', this.setPosition);
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
#canvas {
  border: #2c3e50 1px solid;  
  width: 300px;
  height: 150px;
}
</style>
