<template>
    <div class="image__preview" v-if="image">
        <img :src="image">
        <button class="btn btn__danger image__close" @click="$emit('close')">
            &times;
        </button>
    </div>
</template>

<script type="text/javascript">
    export default {
        props: {
            preview: {
                type: [String, File],
                default: null,
            }
        },
        data() {
            return {
                image: null,
            }
        },
        created() {
            this.setPreview();
        },
        watch: {
            'preview': 'setPreview',
        },
        methods: {
            setPreview() {
                if (this.preview instanceof File) {
                    const fileRender = new FileReader();
                    fileRender.onload = (event) => {
                        this.image = event.target.result;
                    }
                    fileRender.readAsDataURL(this.preview);
                } else if (typeof this.preview === 'string') {
                    this.image = `/images/${this.preview}`;
                } else {
                    this.image = null;
                }
            }
        }
    }
</script>