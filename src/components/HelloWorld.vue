<template>
  <div>
    <h2>{{ isGetting }}</h2>
    <h2>{{ Freq }}</h2>
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
  data: function () {
    return {
      Freq: 0,
      isGetting: false,
    };
  },
  mounted() {
    const analysis_p5 = (_p5) => {
      this.isGetting = false;
      let isRecorded = false;
      let h;
      let pixel_distance = 5;
      let p5 = _p5;
      let status = 0;
      let vol_history = [];
      let peak_history = {};
      let mic, recorder, soundFile, fft, analyzer;
      let isUserStarted = false;
      const fft_bin = 1024;

      p5.setup = () => {
        mic = new P5.AudioIn();
        recorder = new P5.SoundRecorder();
        soundFile = new P5.SoundFile();
        fft = new P5.FFT(0.8, fft_bin);
        analyzer = new P5.Amplitude();

        p5.createCanvas(this.$refs.p5canvas.clientWidth, 300);
        h = p5.height / 2;
        p5.background("white");
        p5.strokeWeight(2);
        p5.stroke("#e56b6f");
        p5.line(p5.width / 2, 0, p5.width / 2, p5.height);

        this.$refs.analysis_button.onclick = () => {
          if (!isUserStarted) {
            p5.userStartAudio().then(() => {
              mic.start();
              isUserStarted = true;
            });
          }
          status = 0;

          toggleMic.bind(this)(() => {
            p5.textSize(50);
            SecCountdown(3, (x) => {
              p5.background("white");
              p5.noStroke();
              p5.textAlign(p5.CENTER);
              p5.text(x, p5.width / 2, p5.height / 2);
            });
            setTimeout(() => {
              recorder.record(soundFile);
              status = 1;
            }, 4000);
          });
        };

        this.$refs.play_button.onclick = () => {
          PlayRec();
        };
      };

      p5.draw = () => {
        // draw rec mode
        if (status === 1) {
          p5.background("white");
          p5.strokeWeight(2);
          p5.stroke("#e56b6f");
          p5.line(p5.width / 2, 0, p5.width / 2, p5.height);
          h = p5.map(mic.getLevel(), 0, 1, p5.height / 2, 0);
          vol_history.push(h);
          p5.stroke("black");
          p5.strokeWeight(pixel_distance / 2);

          p5.beginShape();
          for (let index = 0; index < vol_history.length; index++) {
            if (index % pixel_distance === 0 && vol_history[index] !== -1) {
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

        // draw fft mode
        if (status === 2) {
          p5.strokeWeight(1);
          p5.background("white");
          let spectrum = fft.analyze();

          const ave_spe_amp =
            spectrum.reduce((a, c) => a + c) / spectrum.length;

          const peak_obj_list = FindSpecPeak(spectrum);
          let top3_index = [];
          let top3_spec_amp_ave = 0;

          // 基音がほしいのでTop3のampをfreqで一応昇順sort
          let top3_peak_freq_sorted_obj = peak_obj_list.slice(0, 3);

          top3_peak_freq_sorted_obj.sort(function (a, b) {
            if (a.freq < b.freq) return -1;
            if (a.freq > b.freq) return 1;
            return 0;
          });

          for (
            let index = 0;
            index < top3_peak_freq_sorted_obj.length;
            index++
          ) {
            top3_index.push(top3_peak_freq_sorted_obj[index].key);
            top3_spec_amp_ave =
              top3_peak_freq_sorted_obj[index].value + top3_spec_amp_ave;
          }
          top3_spec_amp_ave = top3_spec_amp_ave / 3;

          // しきい値を設けてピークを取る
          if (
            top3_spec_amp_ave > ave_spe_amp * 2 &&
            analyzer.getLevel() > 0.005
          ) {
            if (top3_index[0].toString() in peak_history) {
              peak_history[top3_index[0].toString()] += 1;
            } else {
              peak_history[top3_index[0].toString()] = 1;
            }
            this.Freq = Math.ceil(
              (p5.sampleRate() * top3_index[0]) / (fft_bin * 2)
            );
            console.log(peak_history);
          }

          p5.beginShape();
          for (let i = 0; i < spectrum.length; i++) {
            p5.stroke("black");
            if (top3_index.includes(i)) {
              p5.stroke("red");
            }
            p5.line(i, p5.height, i, p5.map(spectrum[i], 0, 255, p5.height, 0));
          }
          p5.endShape();
        }
      };

      function SecCountdown(sec, f) {
        setTimeout(() => {
          f(sec);
          sec--;
          if (sec > 0) {
            SecCountdown(sec, f);
          }
        }, 1000);
      }

      function toggleMic(start_callback) {
        if (mic.enabled) {
          if (this.isGetting) {
            recorder.stop();
            this.isGetting = false;
            isRecorded = true;
          } else {
            recorder.setInput(mic);
            this.isGetting = true;
            isRecorded = false;
            vol_history = [];
            vol_history.length = Math.ceil(p5.width / 2);
            vol_history.fill(-1);
            start_callback();
          }
        }
      }

      function PlayRec() {
        if (isRecorded) {
          soundFile.play();
          analyzer.setInput(soundFile);
          fft.setInput(soundFile);
          peak_history = {};
          status = 2;
        }
      }

      function FindSpecPeak(spe) {
        // 600Hzまでの入力をカット
        const cut_index = Math.ceil((600 * fft_bin * 2) / p5.sampleRate());

        // カットしたindex分のズレを修正
        const peak_index = detectPeaks(spe.slice(cut_index)).map(
          (x) => x + cut_index
        );
        // Ampの大きい順に並び替え
        let peak_amp_list = [];
        for (let index = 0; index < peak_index.length; index++) {
          peak_amp_list.push({
            key: peak_index[index],
            value: spe[peak_index[index]],
            freq: (p5.sampleRate() * peak_index[index]) / (fft_bin * 2),
          });
        }
        peak_amp_list.sort(function (a, b) {
          if (a.value > b.value) return -1;
          if (a.value < b.value) return 1;
          return 0;
        });

        return peak_amp_list;
      }

      // https://stackoverflow.com/questions/43567335/finding-peaks-and-troughs-in-time-series-data-in-a-1d-array-javascript
      function detectPeaks(array) {
        const start = 1; // Starting index to search
        const end = array.length - 2; // Last index to search
        let obj = { peaks: [], troughs: [] }; // Object to store the indexs of peaks/thoughs

        for (let i = start; i <= end; i++) {
          let current = array[i];
          let last = array[i - 1];
          let next = array[i + 1];

          if (current > next && current > last) obj.peaks.push(i);
          else if (current < next && current < last) obj.troughs.push(i);
        }
        return obj.peaks;
      }
    };

    new P5(analysis_p5, "canvas");
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
