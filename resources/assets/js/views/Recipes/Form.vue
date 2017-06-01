<template>
    <div class="recipe__show">
        <div class="recipe__header">
            <h3>{{ action }} Recipe</h3>
            <div>
                <button class="btn btn__primary" @click="save" :disabled="isProcessing">
                    Save
                </button>
                <button class="btn" @click="$router.back()" :disabled="isProcessing">
                    Cancel
                </button>
            </div>
        </div>
        <div class="recipe__row">
            <div class="recipe__image">
                <div class="recipe__box">
                    <image-upload v-model="form.image"></image-upload>
                    <small class="error__control" v-if="error.image">
                        {{ error.image[0] }}
                    </small>
                </div>
            </div>
            <div class="recipe__details">
                <div class="recipe__details_inner">
                    <div class="form__group">
                        <label>Name</label>
                        <input class="form__control" type="text" v-model="form.name">
                        <small class="error__control" v-if="error.name">{{ error.name[0] }}</small>
                    </div>
                    <div class="form__group">
                        <label>Description</label>
                        <textarea class="form__control" type="text" v-model="form.description"></textarea>
                        <small class="error__control" v-if="error.description">{{ error.description[0] }}</small>
                    </div>
                </div>
            </div>
        </div>
        <div class="recipe__row">
            <div class="recipe__ingredients">
                <div class="recipe__box">
                    <h3 class="recipe__sub_title">Ingredients</h3>
                    <div class="recipe__form" v-for="(ingredient, index) in form.ingredients">
                        <input type="text" v-model="ingredient.name" class="form__control"
                               :class="error[`ingredients.${index}.name`] ? 'error__bg' : ''">
                        <input type="text" v-model="ingredient.qty" class="form__control form__qty"
                               :class="error[`ingredients.${index}.qty`] ? 'error__bg' : ''">
                        <button class="btn btn__danger" @click="remove('ingredients', index)">
                            &times;
                        </button>
                    </div>
                    <button class="btn" @click="addIngredient">Add Ingredient</button>
                </div>
            </div>
            <div class="recipe__directions">
                <div class="recipe__directions_inner">
                    <h3 class="recipe__sub_title">Directions</h3>
                    <div class="recipe__form" v-for="(direction, index) in form.directions">
                    <textarea type="text" v-model="direction.description" class="form__control form__description"
                              :class="error[`directions.${index}.description`] ? 'error__bg' : ''"></textarea>
                        <button class="btn btn__danger" @click="remove('directions', index)">
                            &times;
                        </button>
                    </div>
                    <button class="btn" @click="addDirection">Add Direction</button>
                </div>

            </div>
        </div>
    </div>
</template>

<script type="text/javascript">
    import Vue from 'vue'
    import Flash from '../../helpers/flash'
    import {get, post} from '../../helpers/api'
    import { toMultipartedForm } from '../../helpers/form'
    import ImageUpload from '../../components/ImageUpload.vue'

    export default {
        components: {
            ImageUpload,
        },
        data() {
            return {
                form: {
                    ingredients: [],
                    directions: [],
                },
                error: {},
                isProcessing: false,
                initializeUrl: `/api/recipes/create`,
                storeUrl: `/api/recipes`,
                action: 'Create',
            }
        },
        created() {
            if (this.$route.meta.mode === 'edit') {
                this.initializeUrl = `/api/recipes/${this.$route.params.id}/edit`;
                this.storeUrl = `/api/recipes/${this.$route.params.id}?_method=PUT`;
                this.action = 'Update';
            }
            get(this.initializeUrl)
                    .then((res) => {
                        Vue.set(this.$data, 'form', res.data.form)
                    })
        },
        methods: {
            save() {
                this.isProcessing = true;
                console.log(this.$route.meta)
                const form = toMultipartedForm(this.form, this.$route.meta.mode);
                post(this.storeUrl, form)
                        .then((res) => {
                            if (res.data.saved) {
                                Flash.setSuccess(res.data.message);
                                this.$router.push(`/recipes/${res.data.id}`)
                            }
                        })
                        .catch((err) => {
                            if (err.response.status == 422) {
                                this.error = err.response.data;
                            }
                            this.isProcessing = false;
                        })
            },
            addDirection() {
                this.form.directions.push({direction: ''})
            },
            addIngredient() {
                this.form.ingredients.push({
                    name: '',
                    qty: ''
                })
            },
            remove(type, index) {
                if (this.form[type].length > 1) {
                    this.form[type].splice(index, 1)
                }
            }
        }
    }
</script>