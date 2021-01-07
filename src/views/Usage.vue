<template>
  <v-container>
    <v-row justify="center" align="center">
      <v-col cols="12">
        <h2>準備</h2>
      </v-col>
      <v-col cols="12" md="6">
        <v-img
          max-height="200px"
          max-width="200px"
          src="../assets/img/Preparation.png"
          class="mr-md-0 mx-auto"
        />
      </v-col>
      <v-col cols="12" md="6">
        <ul class="text-md-left">
          <li>スマホもしくはパソコン</li>
          <li><span class="highlight">割れやすそうな</span>ワイングラス</li>
          <li><span class="highlight">いい感じの長さ</span>のストロー</li>
          <li>できれば<span class="highlight">爆音スピーカー</span></li>
          (最悪スマホスピーカーでもOK)
        </ul>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <h2>破壊</h2>
      </v-col>
      <v-col cols="12">
        <ol class="text-center">
          <li>Analysisボタンを押す</li>
          (マイクの確認ダイアログがでたら許可)
          <li><span class="highlight">もう一度</span>Analysisボタンを押す</li>
          <li>
            <span class="highlight">カウントダウン後</span
            >グラスをはじいて音を出す
          </li>
          <li>サイトが音を自動分析</li>
          <li>Playボタンは破壊音を再生！</li>
        </ol>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <h2>ポイント</h2>
      </v-col>
      <v-col cols="12">
        <ul class="text-center">
          <li><span class="highlight">爆音</span>で破壊音を再生</li>
          <li>音をグラスに<span class="highlight">近づける</span></li>
        </ul>
      </v-col>
    </v-row>
  </v-container>
</template>
<style lang="scss" scoped>
.highlight {
  background: linear-gradient(transparent 50%, #ffff66 0%);
  line-height: 1.3em;
}
</style>
