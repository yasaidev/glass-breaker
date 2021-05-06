<template>
  <div>
    <h1>{{ state_msg[state_id] }}</h1>
    <h2 v-if="state_id === 3">{{ tweaked_Freq }} Hz</h2>

    <div id="canvas" ref="p5canvas"></div>

    <v-container>
      <v-row justify="center" align-content="center">
        <v-col cols="12">
          <v-btn
            large
            ref="analysis_button"
            depressed
            outlined
            v-bind:loading="state_id === 1"
          >
            Analysis🔎
          </v-btn>
        </v-col>
        <v-col cols="12">
          <v-btn
            color="amber"
            large
            depressed
            outlined
            ref="play_button"
            v-bind:disabled="state_id !== 3"
            >{{ isPlay ? "Stop👻" : "Play👽" }}</v-btn
          >
        </v-col>
        <v-col cols="10">
          <v-slider
            max="100"
            min="-100"
            v-model="slider_value"
            v-bind:disabled="state_id !== 3"
            prepend-icon="mdi-waveform"
            append-icon="mdi-reload"
            @click:append="ResetFreq"
          >
          </v-slider>
        </v-col>
        <v-col cols="5">
          <ShareButton :Freq="tweaked_Freq"></ShareButton>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import P5 from "p5/lib/p5.min";
import "../js/p5.sound.min";
import ShareButton from "./Share.vue";

export default {
  name: "AudioAnalysis",
  components: {
    ShareButton,
  },
  data: function () {
    return {
      Freq: 0,
      tweaked_Freq: 0,
      state_msg: ["待機中...", "測定まで...", "測定中...", "共振周波数"],
      state_id: 0,
      isPlay: false,
      slider_value: 0,
    };
  },
  methods: {
    ResetFreq: function () {
      this.slider_value = 0;
    },
  },
  mounted: function () {
    const audio_analysis_p5 = (_p5) => {
      const p5 = _p5;

      const line_distance = 5;
      const fft_bin = 1024;
      let mic, fft, osc;

      let waveform;
      let isUserStarted = false;
      this.isPlay = false;

      let wait_circle_x;
      let peak_history = {};

      // 解析ボタン
      this.$refs.analysis_button.$el.onclick = () => {
        this.$gtag.event("計測ボタン", {
          event_category: "アプリボタンクリック",
          event_label: "Alaysis",
          value: 1,
        });
        if (!isUserStarted) {
          p5.userStartAudio().then(() => {
            mic.start();
            isUserStarted = true;
          });
        } else if (!mic.enalbed) {
          mic.start();
        }

        peak_history = {};
        osc.stop();
        toggleMic(
          this.state_id,
          () => {
            // start callback
            // countdown state
            this.state_id = 1;

            // draw countdown
            p5.stroke("black");
            p5.noStroke();
            p5.textAlign(p5.CENTER);
            p5.textSize(50);
            SecCountdown(3, (x) => {
              p5.background("white");
              p5.text(x, p5.width / 2, p5.height / 2);
            });

            // end countdown and analysis state
            setTimeout(() => {
              fft.setInput(mic);
              this.state_id = 2;
            }, 4000);
          },
          () => {
            // stop callback
            this.state_id = 0;
          }
        );
      };

      this.$refs.play_button.$el.onclick = () => {
        this.$gtag.event("プレイボタン", {
          event_category: "アプリボタンクリック",
          event_label: "Play",
          value: 1,
        });
        if (!isUserStarted || !this.isPlay) {
          p5.userStartAudio().then(() => {
            osc.start();
            isUserStarted = true;
            this.isPlay = true;
          });
        } else if (this.isPlay) {
          this.isPlay = false;
          osc.stop();
        }
      };

      p5.setup = () => {
        mic = new P5.AudioIn();
        fft = new P5.FFT(0.8, fft_bin);
        osc = new P5.Oscillator("sine");

        p5.createCanvas(
          this.$refs.p5canvas.clientWidth,
          this.$refs.p5canvas.clientHeight
        );
        p5.background("white");

        wait_circle_x = WaitCircleIndexInit();
      };

      p5.draw = () => {
        // waiting mode
        if (this.state_id === 0) {
          p5.background("white");
          p5.fill("black");
          p5.beginShape();
          for (let index = 0; index < wait_circle_x.length; index++) {
            if (wait_circle_x[index]) {
              p5.circle(index, p5.height / 2, 10);
            }
          }
          p5.endShape();
          let tmp = wait_circle_x.shift();
          wait_circle_x.push(tmp);
        }

        // state_id === 1はカウントダウンを別関数で記述するのでdraw内はスルー

        // rec mode
        if (this.state_id === 2) {
          DrawWaveform();
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
          if (top3_spec_amp_ave > ave_spe_amp * 2 && mic.getLevel() > 0.01) {
            if (top3_index[0].toString() in peak_history) {
              peak_history[top3_index[0].toString()] += 1;
              if (peak_history[top3_index[0].toString()] > 10) {
                this.state_id = 3;
                this.Freq =
                  Math.round(
                    ((p5.sampleRate() * top3_index[0]) / (fft_bin * 2)) * 10
                  ) / 10;
                this.tweaked_Freq = this.Freq;
                mic.stop();
                fft.setInput(osc);
              }
            } else {
              peak_history[top3_index[0].toString()] = 1;
            }
            this.Freq =
              Math.round(
                ((p5.sampleRate() * top3_index[0]) / (fft_bin * 2)) * 10
              ) / 10;
            this.tweaked_Freq = this.Freq;
          }
        }

        // play mode
        if (this.state_id === 3) {
          this.tweaked_Freq =
            Math.round((this.Freq + this.slider_value / 10) * 10) / 10;
          osc.freq(this.tweaked_Freq);
          DrawWaveform();
        }
      };

      function toggleMic(state_id, start_callback, stop_callback) {
        if (mic.enabled) {
          if (state_id === 1) {
            stop_callback();
          } else {
            start_callback();
          }
        }
      }

      function SecCountdown(sec, f) {
        setTimeout(() => {
          f(sec);
          sec--;
          if (sec > 0) {
            SecCountdown(sec, f);
          }
        }, 1000);
      }

      function WaitCircleIndexInit() {
        let circle_x_index = [];
        circle_x_index.length = p5.width;
        circle_x_index.fill(0);
        return circle_x_index.map((v, i) => {
          if (i % 50 === 0) {
            return 1;
          } else {
            return 0;
          }
        });
      }

      function DrawWaveform(stroke_color = "black") {
        p5.background("white");
        p5.stroke(stroke_color);
        p5.strokeWeight(line_distance / 2);
        waveform = fft.waveform();
        p5.beginShape();
        for (let index = 0; index < waveform.length; index++) {
          let x = p5.map(index, 0, waveform.length, 0, p5.width);
          let y = p5.map(waveform[index], -1, 1, -p5.height / 2, p5.height / 2);
          p5.vertex(x, y + p5.height / 2);
        }
        p5.endShape();
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
    new P5(audio_analysis_p5, "canvas");
  },
};
</script>

<style scoped>
#canvas {
  margin: 0px 10px;
  height: 35vh;
  max-height: 300px;
}
</style>