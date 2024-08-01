<template>
    <v-container>
        <v-row>
            <v-col>
                <v-btn color="success" @click="openCreateModal">Create Task</v-btn>
            </v-col>
        </v-row>
        <v-row>
            <v-col>
                <v-list>
                    <v-list-item v-for="task in tasks" :key="task.id">
                        <v-list-item-content>
                            <v-list-item-title :class="{ 'completed-task': task.is_completed }">{{
                                task.title
                            }}</v-list-item-title>
                            <v-list-item-subtitle :class="{ 'completed-task': task.is_completed }">{{
                                task.description
                            }}</v-list-item-subtitle>
                        </v-list-item-content>
                        <v-list-item-action>
                            <v-layout row>
                                <v-flex class="mr-md-4">
                                    <v-btn color="info" @click="openEditModal(task)">Edit</v-btn>
                                </v-flex>
                                <v-flex class="mr-md-4">
                                    <v-btn color="error" @click="deleteTask(task.id)">Delete</v-btn>
                                </v-flex>
                            </v-layout>
                        </v-list-item-action>
                    </v-list-item>
                </v-list>
            </v-col>
        </v-row>

        <v-dialog v-model="showCreateModal" max-width="500px">
            <v-card>
                <v-card-title>Create Task</v-card-title>
                <v-card-text>
                    <TaskForm @submit="createTask" />
                </v-card-text>
            </v-card>
        </v-dialog>

        <v-dialog v-model="showEditModal" max-width="500px">
            <v-card>
                <v-card-title>Edit Task</v-card-title>
                <v-card-text>
                    <TaskForm :task="editingTask" :isEditing="true" @submit="updateTask" />
                </v-card-text>
            </v-card>
        </v-dialog>
    </v-container>
</template>

<script>
import gql from 'graphql-tag'
import TaskForm from '~/components/TaskForm.vue'

export default {
    components: {
        TaskForm
    },
    async asyncData ({ app }) {
        const { data } = await app.apolloProvider.defaultClient.query({
            query: gql`
        query {
          tasks {
            id
            title
            description
            is_completed
          }
        }
      `
        })
        return { tasks: data.tasks }
    },
    data () {
        return {
            tasks: [],
            showCreateModal: false,
            showEditModal: false,
            editingTask: null
        }
    },
    methods: {
        openCreateModal () {
            this.showCreateModal = true
        },
        openEditModal (task) {
            this.editingTask = { ...task }
            this.showEditModal = true
        },
        async createTask (task) {
            const { data } = await this.$apollo.mutate({
                mutation: gql`
          mutation {
            createTask(title: "${task.title}", description: "${task.description}", is_completed: ${task.is_completed}) {
              id
              title
              description
              is_completed
            }
          }
        `
            })
            this.tasks.push(data.createTask)
            this.showCreateModal = false
        },
        async updateTask (task) {
            const { data } = await this.$apollo.mutate({
                mutation: gql`
          mutation {
            updateTask(id: ${task.id}, title: "${task.title}", description: "${task.description}", is_completed: ${task.is_completed}) {
              id
              title
              description
              is_completed
            }
          }
        `
            })
            const index = this.tasks.findIndex(t => t.id === task.id)
            this.$set(this.tasks, index, data.updateTask)
            this.showEditModal = false
        },
        async deleteTask (id) {
            await this.$apollo.mutate({
                mutation: gql`
          mutation {
            deleteTask(id: ${id})
          }
        `
            })
            this.tasks = this.tasks.filter(task => task.id !== id)
        }
    }
}
</script>

<style>
.completed-task {
    text-decoration: line-through;
}
</style>
