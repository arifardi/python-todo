<template>
  <q-page class="flex flex-center column q-mx-auto" style="width: 500px">
    <div class="text-center col-shrink q-mt-xl">Welcome <span v-if="!tasks.length">, Add a todo to get started</span></div>
    <div class="col-shrink full-width">
      <q-input dense v-model="form.title" label="Title" stack-label />
    </div>
    <div class="col-shrink full-width q-mt-md">
      <q-input dense type="textarea" v-model="form.description" autogrow label="Description" stack-label />
    </div>
    <div class="col-shrink q-mt-md">
      <q-btn label="Add Todo" color="primary" @click="addTodo" no-caps />
    </div>
    <div class="col-shrink q-mt-md">
      List of Todos
    </div>
    <div class="col-shrink q-mt-md" v-for="task in tasks" :key="task.id">
      <q-card :style="{
        width: '400px',
        'bg-primary': task.done ? true : false
      }" class="q-pa-md">
        <q-card-section class="text-h6">
          {{task.title}}
        </q-card-section>
        <q-card-section>
          <div class="row">
            <div class="col-grow">
              description: {{task.description}}
            </div>
            <div class="col-shrink">
              <q-btn @click="mark(task)" dense label="mark as done" no-caps color="secondary"/>
            </div>
          </div>
          <div class="row">
          </div>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn @click="deleteTodo(task.id)" flat icon="delete" no-caps color="red" />
          <q-btn @click="openModal(task.id)" flat icon="edit" no-caps color="primary" />
        </q-card-actions>
      </q-card>
    </div>
    <q-dialog v-model="modal" class="bg-white">
     <div class="bg-white q-pa-lg" style="width: 500px">
       <div class="text-center col-shrink">Update Todo</div>
       <div class="col-shrink full-width">
         <q-input dense v-model="form.title" label="Title" stack-label />
       </div>
       <div class="col-shrink full-width q-mt-md">
         <q-input dense type="textarea" v-model="form.description" autogrow label="Description" stack-label />
       </div>
       <div class="col-shrink q-mt-md">
         <q-btn label="Update Todo" color="primary" @click="updateTodo" no-caps />
       </div>
     </div>
    </q-dialog>
  </q-page>
</template>

<script>
const api = 'http://localhost:5000/' // change according to ngrock
export default {
  name: 'PageIndex',
  data () {
    return {
      tasks: [],
      form: {
        id: '',
        title: '',
        description: '',
        done: false
      },
      modal: false
    }
  },
  watch: {
    modal (value) {
      if (!value) {
        this.form = {
          id: '',
          title: '',
          description: '',
          done: false
        }
      }
    }
  },
  methods: {
    getTodos () {
      this.$axios.get(api + 'todo')
        .then(({ data }) => {
          console.log('data', data)
          this.tasks = data.tasks
        })
        .catch(err => {
          console.log('error', err)
        })
    },
    addTodo () {
      this.$axios.post(api + 'todo', {
        title: this.form.title,
        description: this.form.description
      })
        .then(() => {
          this.getTodos()
        })
        .catch(err => {
          console.log('error', err)
        })
    },
    deleteTodo (id) {
      this.$axios.delete(`${api}todo/${id}`)
        .then(() => {
          this.getTodos()
        })
        .catch(err => {
          console.log('error', err)
        })
    },
    mark (task) {
      task.done = !task.done
      console.log('tasl', task)
      this.$axios.put(`${api}todo/${task.id}`, { task })
        .then(() => {
          this.getTodos()
        })
        .catch(err => {
          console.log('error', err)
        })
    },
    updateTodo () {
      this.$axios.put(`${api}todo/${this.form.id}`, { ...this.form })
        .then(() => {
          this.getTodos()
          this.modal = false
        })
        .catch(err => {
          console.log('error', err)
        })
    },
    openModal (id) {
      this.$axios.get(`${api}todo/${id}`)
        .then(({ data }) => {
          console.log('data', data)
          this.form = data.task
          this.modal = true
        })
        .catch(err => {
          console.log('error', err)
        })
    }
  },
  mounted () {
    this.getTodos()
  }
}
</script>
