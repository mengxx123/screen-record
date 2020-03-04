<template>
    <my-page title="屏幕录制" :page="page">
        <div class="common-container container">
            <video autoplay="true" id="video" controls="true" controlslist="nodownload"></video>

            <div class="btns">
                <ui-raised-button class="btn" primary label="录制" @click="record" v-if="recordVisible" />
                <ui-raised-button class="btn" secondary label="停止" @click="stop" v-if="stopVisible" />
                <ui-raised-button class="btn" label="下载" @click="download" v-if="downloadVisible" />
            </div>

            <div class="created"></div>
        </div>
  </my-page>
</template>

<script>
/* eslint-disable */
export default {
    data() {
        return {
            recordVisible: true,
            stopVisible: false,
            downloadVisible: false,
            page: {
                menu: [
                    // {
                    //     type: "icon",
                    //     icon: "search",
                    //     href: "https://search.yunser.com?utm_source=edu",
                    //     target: "_blank",
                    //     title: "搜索"
                    // },
                    {
                        type: "icon",
                        icon: "apps",
                        href: "https://app.yunser.com?utm_source=edu",
                        target: "_blank",
                        title: "应用"
                    }
                ]
            }
        }
    },
    mounted() {
        this.init()
    },
    methods: {
        init() {
            const video = document.getElementById('video')
            this.video = video
            let recorder
            let _this = this
        
            async function record() {
                let captureStream
        
                try {
                    captureStream = await navigator.mediaDevices.getDisplayMedia({
                        video: true,
                        // audio: true,   not support
                        cursor: 'always'
                    })
                }catch(e){
                    alert('无法获取摄像头数据')
                    return
                }

                _this.downloadVisible = false
                _this.stopVisible = true
                _this.recordVisible = false
        
                window.URL.revokeObjectURL(video.src)
        
                video.autoplay = true
                video.srcObject = captureStream
        
                recorder = new MediaRecorder(captureStream)
                this.recorder = recorder
                recorder.start()
        
                captureStream.getVideoTracks()[0].onended = () => {
                    recorder.stop()
                }
        
                recorder.addEventListener("dataavailable", event => {
                    let videoUrl = URL.createObjectURL(event.data, {type: 'video/ogg'})
            
                    video.srcObject = null
                    video.src = videoUrl
                    video.autoplay = false

                    _this.downloadVisible = true
                    _this.stopVisible = false
                    _this.recordVisible = true
                })
            }

            this._record = record
        },
        record() {
            this._record()
        },
        stop() {
            let tracks = this.video.srcObject.getTracks()
            tracks.forEach(track => track.stop())
            this.recorder.stop()
        },
        download() {
            const url = this.video.src
            const name = new Date().toISOString().slice(0, 19).replace('T',' ').replace(" ","_").replace(/:/g,"-")
            const a = document.createElement('a')
    
            a.style = 'display: none'
            a.download = `${name}.ogg`
            a.href = url
    
            document.body.appendChild(a)
    
            a.click()
        },
    }
};
</script>


<style lang="scss" scoped>
@import "../scss/var";
.btns {
    .btn {
        margin-right: 8px;
    }
}
.container {
    font-family: Arial;
    margin: 4vh auto;
    width: 90vw;
    max-width: 600px;
    text-align: center;
}
#controls {
    text-align: center;
}
#video {
  margin-top: 10px;
  margin-bottom: 20px;
  border: 12px solid #a5adb0;
  border-radius: 15px;
  outline: none;
  width: 100%;
  height: 400px;
  background-color: black;
}
.created {
  color: lightgrey;
  letter-spacing: -0.7px;
  font-size: 1em;
  margin-top: 40px;
}
.created > a {
  color: #4b7bec;
  text-decoration: none;
}
</style>
