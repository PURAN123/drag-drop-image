<template>
    <div class="main">
        <div
            class="dropzone-container"
            @dragover="dragover"
            @dragleave="dragleave"
            @drop="drop"
            :style="isDragging && 'border-color: green;'"
        >
        <div v-if="!uploading">
            <input
            type="file"
            multiple
            name="file"
            id="fileInput"
            class="hidden-input"
            @change="onChange"
            ref="file"
            accept=".pdf,.jpg,.jpeg,.png"
            />
    
            <label for="fileInput" class="file-label">
            <div v-if="isDragging">Release to drop files here.</div>
            <div v-else>Drop files here or <u>click here</u> to upload.</div>
            </label>
        </div>
        <div v-else>
            <p>Uploading...</p>
            <div class="progress-bar">
                <div class="progress" :style="{ width: progress + '%' }"></div>
            </div>
        </div>
            <div class="preview-container mt-4" v-if="files.length">
                <div v-for="file in files" :key="file.name" class="preview-card">
                    <div>
                        <img class="preview-img"  :src=generateURL(file) />
                        <p>
                            {{ file.name }}
                            {{ Math.round(file.size / 1000) + "kb" }}
                        </p>
                    </div>
                    <div>
                        <button
                            class="ml-2"
                            type="button"
                            @click="remove(files.indexOf(file))"
                            title="Remove file"
                        >
                            <b>×</b>
                        </button>
                    </div>
                </div>
            </div>
        </div>
        <div style="margin: 20px;">
            <button style="padding: 20px 30px; font-size: 20px;cursor: pointer;" @click="uploadImages">Upload</button>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            isDragging: false,
            files: [],
            uploading:false,
            progress: 0
        };
    },
    methods: {
        onChange() {
            const self = this;
            let incomingFiles = Array.from(this.$refs.file.files);
            const fileExist = self.files.some((r) =>
                incomingFiles.some(
                (file) => file.name === r.name && file.size === r.size
                )
            );
            if (fileExist) {
                self.showMessage = true;
                alert("New upload contains files that already exist");
            } else {
                self.files.push(...incomingFiles);
            }
        },
        dragover(e) {
            e.preventDefault();
            this.isDragging = true;
        },
        dragleave() {
            this.isDragging = false;
        },
        drop(e) {
            e.preventDefault();
            this.$refs.file.files = e.dataTransfer.files;
            this.onChange();
            this.isDragging = false;
        },
        remove(i) {
            this.files.splice(i, 1);
        },
        generateURL(file) {
            let fileSrc = URL.createObjectURL(file);
            setTimeout(() => {
                URL.revokeObjectURL(fileSrc);
            }, 1000);
            return fileSrc;
        },
        uploadImages() {
            this.uploading=true
            const totalSize = Array.from(this.files).reduce((acc, file) => acc + file.size, 0);
            let uploadedSize = 0;
            const uploadProgress = setInterval(() => {
                if (uploadedSize >= totalSize) {
                    clearInterval(uploadProgress);
                    this.uploading = false;
                    this.progress = 0;
                    this.files=[];
                    return;
                }
                uploadedSize += 100000; // Simulated chunk size
                this.progress = (uploadedSize / totalSize) * 100;
            }, 500);

            this.uploading=true
            // this.$http.post('/upload.php', fd, {
            //     progress(e) {
            //         if (e.lengthComputable) {
            //             this.progress = (e.loaded / e.total) * 100;
            //         }
            //     }
            // });
        }

    },
};
</script>
<style scoped src="@/assets/dropfile.css"></style>