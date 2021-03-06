<template>
    <div>
        <form enctype="multipart/form-data" novalidate>
            <div class="az-drop-file"
                 :style="{'height': height}"
                 @drop.prevent="onDropFiles($event.dataTransfer.items)"
                 @dragover.prevent="">
                <input id="azFileSelector" type="file" :multiple="multiple" class="input-file"
                       :name="uploadFieldName"
                       :accept="accept"
                       @change="onSelectFiles($event.target.files)"/>
                <div class="center">
                    <slot></slot>
                </div>
            </div>
        </form>
    </div>
</template>

<script>

    export default {
        props: {
            accept: {
                type: String,
                default: '*'
            },
            repository: {
                type: String,
                required: true
            },
            thumbnail: {
                type: Boolean,
                default: false
            },
            height: {
                type: String,
                default: '200px'
            },
            multiple: {
                type: Boolean,
                default: true
            }
        },
        data() {
            return {
                uploadFieldName: 'file'
            }
        },
        methods: {
            calcFileSize(bytes, unit) {
                const k = 1024
                return parseFloat((bytes / Math.pow(k, unit)).toFixed(2))
            },
            createFormData(file) {
                const formData = new FormData()
                formData.append(this.uploadFieldName, file)
                formData.append('repository', this.repository)
                formData.append('thumbnail', this.thumbnail)
                return formData
            },
            createPayload(file) {
                return {
                    formData: this.createFormData(file),
                    filename: file.name
                }
            },
            getMaxSizeConfig() {
                const fileConfig = this.$store.state.loki.file
                const maxSize = fileConfig.maxSize.replace(/([A-Za-z]+)/g, '')
                const unit = ['B', 'KB', 'MB', 'GB', 'TB'].indexOf(fileConfig.maxSize.replace(/\d+/g, '').toUpperCase())
                return {maxSize, unit}
            },
            isFileBiggerThanExpected(file) {
                const {maxSize, unit} = this.getMaxSizeConfig()
                const fileSize = this.calcFileSize(file.size, unit)
                return fileSize > maxSize
            },
            onDropFiles(eventItems) {
                if (!Object.keys(eventItems).length) {
                    return;
                }
                const files = []
                Object.keys(eventItems).forEach((key) => {
                    const item = eventItems[key]
                    if (item.kind === 'file') {
                        files.push(item.getAsFile())
                    }
                })
                this.uploadFiles(files)
            },
            onSelectFiles(fileList) {
                if (!fileList.length) {
                    return;
                }
                this.uploadFiles(fileList)
                this.resetSelectedFiles()
            },
            openFileSelector() {
                document.getElementById('azFileSelector').click()
            },
            uploadFiles(fileList) {
                Array
                    .from(Array(fileList.length).keys())
                    .map(x => {
                        const file = fileList[x]
                        if (this.isFileBiggerThanExpected(file)) {
                            this.throwFileExceedMaxLimitSizeEvent(file)
                        } else {
                            const payload = this.createPayload(file)
                            this.$store.dispatch('uploadFile', payload)
                        }
                    });
            },
            resetSelectedFiles() {
                document.getElementById('azFileSelector').value = ''
            },
            throwFileExceedMaxLimitSizeEvent(file) {
                this.$emit('error', {
                    type: 'FILE_EXCEEDED_MAX_SIZE_LIMIT',
                    filename: file.name
                })
            }
        }
    }
</script>

<style scoped lang="stylus">
    .az-drop-file
        border: 2px dashed #ccc
        margin: 10px 0
        padding: 30px 0 20px
        align-content: center
        align-items: center
        text-align: center
        background-color: #eeeeee
        color: #7b7b7b
        display: flex
        position: relative

        &:hover
            border-color: #939393

        .center
            margin: 0 auto
            padding: 0

        .input-file
            opacity: 0
            position: absolute
            left: 0
            top: 0
            width: 10px
            height: 10px
            z-index: 1

        &__big
            padding: 70px 0 60px !important

        p
            width: 100%
            color: #777
            font-size: 13px
            margin-bottom: 0
            margin-top: 15px

        a
            background-color: #d28a2C
            color: white
            padding: 10px 15px 10px 10px
            border-radius: 2px
            font-size: 13px
            &:hover
                background-color: lighten(#d28a2C, 10%)
                border: 1px solid lighten(#d28a2C, 10%) !important

            i
                color: white !important
                position: relative
                top: 2px
                font-size: 18px
                margin-right: 5px
</style>
