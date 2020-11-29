<template>
  <div>
    <div id="canvas" ref="p5canvas"></div>
    <div class="button_parent">
      <div class="button_child">
        <button ref="analysis_button">analysis</button>
      </div>
      <div class="button_child">
        <button ref="play_button">play</button>
      </div>
    </div>
  </div>
</template>
<script>
import P5 from "p5";
import "../js/p5.sound";
import "p5/lib/addons/p5.dom";

export default {
  name: "HelloWorld",
  mounted() {
    const analysis_p5 = (_p5) => {
      let isGetting = false;
      let h;
      let pixel_distance = 5;
      let p5 = _p5;
      const timer = 3;
      let now_time = 0;
      let status = 0;
      let vol_history = [];

      let countdown;
      let mic, recorder, soundFile;
      p5.setup = () => {
        mic = new P5.AudioIn();
        recorder = new P5.SoundRecorder();
        soundFile = new P5.SoundFile();

        p5.createCanvas(this.$refs.p5canvas.clientWidth, 300);
        this.$refs.analysis_button.onclick = toggleMic;
        h = p5.height / 2;
      };
      p5.draw = () => {
        if (isGetting) {
          p5.background("white");
          if (status === 0) {
            countdown = p5.ceil((now_time - p5.millis()) / 1000);
            p5.textSize(50);
            p5.text(countdown, p5.width / 2, p5.height / 2);
            if (countdown === 0) {
              status = 1;
              recorder.record(soundFile);
            }
          } else {
            p5.stroke("#e56b6f");
            p5.line(
              p5.width / 2,
              p5.height / 2 + (p5.height / 2) * 0.8,
              p5.width / 2,
              p5.height / 2 - (p5.height / 2) * 0.8
            );
            const vol = mic.getLevel();
            h = p5.map(vol, 0, 1, p5.height / 2, 0);
            vol_history.push(h);

            p5.stroke("black");
            p5.strokeWeight(pixel_distance / 2);
            p5.beginShape();

            for (let index = 0; index < vol_history.length; index++) {
              if (index % pixel_distance === 0) {
                p5.line(
                  index,
                  vol_history[index],
                  index,
                  p5.height - vol_history[index]
                );
              }
            }
            p5.endShape();

            if (vol_history.length + p5.width / 2 > p5.width) {
              vol_history.splice(0, pixel_distance);
            }
          }
        }
      };

      p5.touchStarted = () => {
        p5.userStartAudio();
      };

      function toggleMic() {
        if (isGetting) {
          // AudioInオブジェクトをオフにする
          mic.stop();
          recorder.stop();
          soundFile.play();
          isGetting = false;
        } else {
          mic.start(() => {
            recorder.setInput(mic);
            now_time = p5.millis() + timer * 1000;
            status = 0;
            isGetting = true;
            vol_history = [];
            vol_history.length = p5.width / 2;
          });
        }
      }
    };

    new P5(analysis_p5, "canvas");
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
