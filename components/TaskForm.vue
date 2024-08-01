<template>
    <v-form ref="form" v-model="valid">
        <v-text-field v-model="task.title" label="Title" :rules="[rules.required]" required></v-text-field>
        <v-textarea v-model="task.description" label="Description"></v-textarea>
        <v-checkbox v-model="task.is_completed" label="Is Completed"></v-checkbox>
        <v-btn :disabled="!valid" color="primary" @click="submit">{{ isEditing ? 'Update' : 'Create' }}</v-btn>
    </v-form>
</template>

<script>
export default {
    props: {
        task: {
            type: Object,
            default: () => ({
                title: '',
                description: '',
                is_completed: false
            })
        },
        isEditing: {
            type: Boolean,
            default: false
        }
    },
    data () {
        return {
            valid: false,
            rules: {
                required: value => !!value || 'Required.'
            }
        }
    },
    methods: {
        submit () {
            if (this.$refs.form.validate()) {
                this.$emit('submit', this.task)
            }
        }
    }
}
</script>
