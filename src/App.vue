<template>
  <div id="app">
    <div class="videos">
      <video ref="video1" playsinline autoplay />
      <video ref="video2" playsinline autoplay />
    </div>
    <div class="res">
      <div v-for="key in Object.keys(tests)" :key="key" :class="[tests[key]]">
        <span></span> {{ $options.labels[key] }}
      </div>
    </div>
  </div>
</template>

<script>
const labels = {
  test1: 'Create first mediaStream with 640 width, create another mediaStream with 1280 width',
  test2: 'Create first mediaStream with 1280 width, create another mediaStream with 640 width',
  test3: 'Create first mediaStream with 640 width, clone, applyConstraints to 1280 width',
  test4: 'Create first mediaStream with 1280 width, clone, applyConstraints to 640 width',
  test5: 'Create first mediaStream, disable tracks, clone, enable tracks on clone',
  test6: 'Create first mediaStream width 1280 width, release it, create mediaStream with 640width, then recreate media with 1280px'
}

export default {
  labels,
  data () {
    return {
      tests: {
        test1: 'pending',
        test2: 'pending',
        test3: 'pending',
        test4: 'pending',
        test5: 'pending',
        test6: 'pending'
      }
    }
  },
  async mounted () {
    await this.test1()
    await this.test2()
    await this.test3()
    await this.test4()
    await this.test5()
    await this.test6()
  },
  methods: {
    async createMediaStream (ideal) {
      const constraints = { video: { width: { ideal } } }
      return navigator.mediaDevices.getUserMedia(constraints)
    },
    releaseStream (stream) {
      const tracks = stream && stream.getTracks()
      tracks && tracks.forEach(track => track.stop())
    },
    async test1 () {
      const mediaStream640 = await this.createMediaStream(640)
      const mediaStream1280 = await this.createMediaStream(1280)

      const track640 = mediaStream640.getVideoTracks()[0]
      const track1280 = mediaStream1280.getVideoTracks()[0]
      const settings640 = await track640.getSettings()
      const settings1280 = await track1280.getSettings()
      const success = settings640.width === 640 && settings1280.width === 1280
      console.log('640', settings640.width, '1280', settings1280.width)
      this.tests.test1 = success ? 'success' : 'fail'

      this.releaseStream(mediaStream640)
      this.releaseStream(mediaStream1280)
    },
    async test2 () {
      const mediaStream1280 = await this.createMediaStream(1280)
      const mediaStream640 = await this.createMediaStream(640)
      const track1280 = mediaStream1280.getVideoTracks()[0]
      const track640 = mediaStream640.getVideoTracks()[0]
      const settings1280 = await track1280.getSettings()
      const settings640 = await track640.getSettings()
      const success = settings640.width === 640 && settings1280.width === 1280
      console.log('640', settings640.width, '1280', settings1280.width)
      this.tests.test2 = success ? 'success' : 'fail'

      this.releaseStream(mediaStream640)
      this.releaseStream(mediaStream1280)
    },
    async test3 () {
      const mediaStream640 = await this.createMediaStream(640)
      const mediaStream640Cloned = mediaStream640.clone()
      const clonedTrack = mediaStream640Cloned.getVideoTracks()[0]
      await clonedTrack.applyConstraints({ width: { ideal: 1280 } })
      const clonedSettings = await clonedTrack.getSettings()
      const track640 = mediaStream640.getVideoTracks()[0]
      const settings640 = await track640.getSettings()
      const success = settings640.width === 640 && clonedSettings.width === 1280
      console.log('640', settings640.width, '1280', clonedSettings.width)
      this.tests.test3 = success ? 'success' : 'fail'

      this.releaseStream(mediaStream640)
      this.releaseStream(mediaStream640Cloned)
    },
    async test4 () {
      const mediaStream1280 = await this.createMediaStream(1280)
      const mediaStream1280Cloned = mediaStream1280.clone()
      const clonedTrack = mediaStream1280Cloned.getVideoTracks()[0]
      await clonedTrack.applyConstraints({ width: { ideal: 640 } })
      const clonedSettings = await clonedTrack.getSettings()
      const track1280 = mediaStream1280.getVideoTracks()[0]
      const settings1280 = await track1280.getSettings()
      const success = settings1280.width === 1280 && clonedSettings.width === 640
      console.log('1280', settings1280.width, '640', clonedSettings.width)
      this.tests.test4 = success ? 'success' : 'fail'

      this.releaseStream(mediaStream1280)
      this.releaseStream(mediaStream1280Cloned)
    },
    async test5 () {
      const mediaStream = await this.createMediaStream(1280)
      this.$refs.video1.srcObject = mediaStream
      await this.$refs.video1.play()
      mediaStream.getVideoTracks().forEach(track => (track.enabled = false))
      const mediaStreamCloned = mediaStream.clone()
      this.$refs.video2.srcObject = mediaStreamCloned
      await this.$refs.video2.play()
      mediaStreamCloned.getVideoTracks().forEach(track => (track.enabled = true))
      await this.$refs.video2.play()
      const originalTrack = mediaStream.getVideoTracks()[0]
      const clonedTrack = mediaStreamCloned.getVideoTracks()[0]
      const success = originalTrack.enabled === false && clonedTrack.enabled === true
      this.tests.test5 = success ? 'success' : 'fail'

      this.releaseStream(mediaStream)
      this.releaseStream(mediaStreamCloned)
    },
    async test6 () {
      let mediaStream1280 = await this.createMediaStream(1280)

      // this.releaseStream(mediaStream1280)
      // => fail

      const mediaStream640 = await this.createMediaStream(640)
      const track640 = mediaStream640.getVideoTracks()[0]
      const settings640 = await track640.getSettings()

      this.releaseStream(mediaStream1280)

      mediaStream1280 = await this.createMediaStream(1280)
      const track1280 = mediaStream1280.getVideoTracks()[0]
      const settings1280 = await track1280.getSettings()

      const success = settings640.width === 640 && settings1280.width === 1280
      console.log('640', settings640.width, '1280', settings1280.width)
      this.tests.test6 = success ? 'success' : 'fail'

      this.releaseStream(mediaStream640)
      this.releaseStream(mediaStream1280)
    }
  }
}
</script>

<style scoped lang="less">
.videos {
  display: flex;

  video {
    border: 1px solid;
    width: 50%
  }
}

.res {
  > div {
    display: flex;
    align-items: center;

    span {
      display: inline-block;
      width: 50px;
      height: 50px;
    }

    &.pending span {
      background: lightgray;
    }

    &.success span {
      background: limegreen;
    }

    &.fail span {
      background: tomato;
    }
  }
}
</style>
