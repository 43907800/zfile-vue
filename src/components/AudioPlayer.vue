<template>
    <aplayer v-show="fileList.length > 0 && audioIndex !== -1" ref="aplayer" id="aplyer" fixed
             :audio="fileList" @listSwitch="updateCover" loop='none'/>
</template>

<script>
    export default {
        name: "AudioPlayer",
        data() {
            return {
                audioInfo: {}
            }
        },
        props: {
            fileList: Array,
            audioIndex: Number
        },
        methods: {
            updateCover() {
                let currentMusic = this.$refs.aplayer.currentMusic;
                
                let filePath = currentMusic.path + currentMusic.name;
                document.title = currentMusic.name;
                if(this.fileList.length > 0 && this.audioIndex !== -1){
                    this.common.saveRecord(currentMusic.path,currentMusic.name)
                }
                if (currentMusic.updateSrc) return;
                let driveId = this.common.getDriveId();

                this.$http.get('common/audio-info', {params: {url: currentMusic.url, filePath: filePath, driveId: driveId}}).then((response) => {
                    let data = response.data.data;
                    let el = this.$refs.aplayer.$el;
                    if (data.src != null && data.src != ''){
                        this.$refs.aplayer.currentMusic.url = data.src;
                        this.$refs.aplayer.currentMusic.updateSrc = true;
                        console.log('更改src')
                    }
                    el.getElementsByClassName('aplayer-pic')[0].style.backgroundImage = 'url(' + data.cover + ')';
                    el.getElementsByClassName('aplayer-list-light')[0].getElementsByClassName('aplayer-list-author')[0].innerHTML = data.artist
                })
            }
        },
        watch: {
            audioIndex() {
                if (this.fileList.length > 0 && this.audioIndex !== -1) {
                    this.$refs.aplayer.play();
                    this.$refs.aplayer.switch(this.audioIndex);
                } else {
                    this.$refs.aplayer.pause();
                }
            },
            fileList(data) {
                if (data.length === 0) {
                    this.$refs.aplayer.pause();
                } else {
                    for (let file of this.fileList) {
                        file.artist = '';
                        file.cover = '';
                    }
                }
            }
        },
        mounted() {
            let el = this.$refs.aplayer.$el;
            el.getElementsByClassName("aplayer-miniswitcher")[0].click();
            el.getElementsByClassName('aplayer-icon-order')[0].setAttribute('title', '播放顺序');
            el.getElementsByClassName('aplayer-icon-loop')[0].setAttribute('title', '循环模式');
            el.getElementsByClassName('aplayer-icon-menu')[0].setAttribute('title', '播放列表');
        }
    }
</script>

<style scoped>
    #aplyer >>> .el-icon-close {
        position: absolute;
        right: 0;
        top: 0;
    }
</style>