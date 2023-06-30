<!--
 * @Author: RyanWilson
 * @Date: 2023-06-30 13:17:09
-->
<template>
  <canvas id="drawing" tabindex="0"></canvas>
</template>


<script>
export default {
  data() {
    return {
      canvas_ctx: null,
      canvas: null,
      boundary: 5,
      scale: 40,
      width: 1024,
      height: 512,
      dot_x1: -1,
      dot_x2: 1,
      dot_y: 1,
      active_dot: 0,
    };
  },
  mounted() {
    this.init_canvas();
    this.init_listen_mouse();
  },
  methods: {
    draw_line(src_x, src_y, dst_x, dst_y, immediately) {
      if (!this.canvas || !this.canvas_ctx) {
        return;
      }
      //TODO:Determine whether the coordinates are legal

      if (src_x >= 0 && src_y >= 0) {
        this.canvas_ctx.beginPath();
        this.canvas_ctx.moveTo(src_x, src_y);
      }

      this.canvas_ctx.lineTo(dst_x, dst_y);
      if (immediately || immediately == undefined) {
        this.canvas_ctx.stroke();
      }
    },
    cacu_distance(x1, y1, x2, y2) {
      const dx = x2 - x1;
      const dy = y2 - y1;
      const distance = Math.sqrt(dx * dx + dy * dy);
      return distance;
    },
    get_dots_pos() {
      //x-axis dot 1
      let dx1 = this.dot_x1 * this.scale + this.width / 2;
      let dy1 = this.height / 2;
      //x-axis dot 2
      let dx2 = this.dot_x2 * this.scale + this.width / 2;
      let dy2 = this.height / 2;
      //y-axis dot
      let dx3 = this.width / 2;
      let dy3 = this.height / 2 - this.dot_y * this.scale;

      return { dx1, dy1, dx2, dy2, dx3, dy3 };
    },
    init_canvas() {
      let canvas = document.getElementById("drawing");
      canvas.width = this.width;
      canvas.height = this.height;

      this.canvas = canvas;
      this.canvas_ctx = canvas.getContext("2d");
      this.canvas_ctx.strokeStyle = "#000";
      this.canvas_ctx.font = "14px Arial";

      //draw x-axis
      this.draw_line(0, this.height / 2, this.width, this.height / 2);
      //draw y-axis
      this.draw_line(this.width / 2, 0, this.width / 2, this.height);

      //draw x-axis direction arrow
      this.draw_line(
        this.width - 10,
        this.height / 2 - 10,
        this.width,
        this.height / 2,
        false
      );
      this.draw_line(-1, -1, this.width - 10, this.height / 2 + 10, true);
      //draw y-axis direction arrow
      this.draw_line(this.width / 2 - 10, 10, this.width / 2, 0, false);
      this.draw_line(-1, -1, this.width / 2 + 10, 10, true);

      //draw X-scale
      for (
        let i = 1, value = this.scale;
        value <= this.width / 2;
        i++, value += this.scale
      ) {
        let pos_x1 = this.width / 2 + value;
        let pos_x2 = this.width / 2 - value;
        let pos_y = this.height / 2 + 20;

        // this.canvas_ctx.fillText(i, pos_x, pos_y);
        this.canvas_ctx.fillText(i, pos_x1, pos_y);
        this.canvas_ctx.fillText(-i, pos_x2, pos_y);
      }
      //draw Y-scale
      for (
        let i = 1, value = this.scale;
        value <= this.height / 2;
        i++, value += this.scale
      ) {
        let pos_x = this.width / 2 + 20;
        let pos_y1 = this.height / 2 - value;
        let pos_y2 = this.height / 2 + value;

        // this.canvas_ctx.fillText(i, pos_x, pos_y);
        this.canvas_ctx.fillText(i, pos_x, pos_y1);
        this.canvas_ctx.fillText(-i, pos_x, pos_y2);
      }

      //finnaly
      this.draw_triangle();
    },
    init_listen_mouse() {
      var debounceInterval = 10;
      var timerId;

      // limit motion listener call frequency
      this.canvas.addEventListener("mousemove", (e) => {
        clearTimeout(timerId);
        timerId = setTimeout(() => {
          if (this.active_dot > 0) {
            let pos_x = e.clientX;
            let pos_y = e.clientY;
            //clear line before update 
            this.draw_clear_triangle();

            switch (this.active_dot) {
              case 1: {
                this.dot_x1 = (pos_x - this.width / 2) / this.scale;
                break;
              }
              case 2: {
                this.dot_x2 = (pos_x - this.width / 2) / this.scale;
                break;
              }
              case 3: {
                this.dot_y = (this.height / 2 - pos_y) / this.scale;
                break;
              }
            }
            this.draw_triangle();
          }
        }, debounceInterval);
      });

      this.canvas.addEventListener("mouseup", (e) => {
        console.log("u have released dot " + this.active_dot);
        this.active_dot = 0;
      });

      this.canvas.addEventListener("mousedown", (e) => {
        let pos_x = e.clientX;
        let pos_y = e.clientY;

        let dots = this.get_dots_pos();

        //caculet the most closed dot
        let d_x1 = this.cacu_distance(pos_x, pos_y, dots.dx1, dots.dy1);
        let d_x2 = this.cacu_distance(pos_x, pos_y, dots.dx2, dots.dy2);
        let d_y = this.cacu_distance(pos_x, pos_y, dots.dx3, dots.dy3);

        let min = Math.min(d_x1, d_x2, d_y);

        if (min <= 5) {
          if (min === d_x1) {
            this.active_dot = 1;
          } else if (min === d_x2) {
            this.active_dot = 2;
          } else {
            this.active_dot = 3;
          }

          console.log("u have pressed dot " + this.active_dot);
        }
      });
    },

    draw_triangle() {
      let dots = this.get_dots_pos();

      this.draw_line(dots.dx1, dots.dy1, dots.dx3, dots.dy3, false);
      this.draw_line(-1, -1, dots.dx2, dots.dy2, true);
    },
    draw_clear_triangle() {
      //TODO:clear rect
      this.canvas_ctx.strokeStyle = "#fff";
      this.draw_triangle();
      this.canvas_ctx.strokeStyle = "#000";
    },
  },
};
</script>

<style>
</style>

