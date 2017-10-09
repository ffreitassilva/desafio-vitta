<template>
  <q-modal ref="taskManagerModal" :content-css="{minWidth: '30vw', minHeight: '500px'}">
    <q-modal-layout>

      <q-toolbar slot="header">
        <div class="q-toolbar-title">
          Tarefa
        </div>
        <q-btn flat @click="$refs.taskManagerModal.close()">
          <q-icon name="close" />
        </q-btn>
      </q-toolbar>

      <q-toolbar color="white" slot="footer" class="justify-end">
        <q-btn class="bg-primary" @click="saveTask">Confirmar</q-btn>
      </q-toolbar>

      <div class="layout-padding">
        <!-- Description -->
        <q-field
          :error="$v.task.description.$error"
          error-label="* campo obrigatório!"
        >
          <q-input
            type="text"
            float-label="Descrição da tarefa"
            v-model="task.description"
            @blur="$v.task.description.$touch"
          />
        </q-field>

        <!-- Happen Time -->
        <q-field
          :error="$v.task.happenTime.$error"
          error-label="* campo obrigatório!"
        >
          <q-datetime 
            type="datetime"
            float-label="Data e hora que a tarefa acontecerá"
            format="DD/MM/YYYY HH:mm"
            format24h
            v-model="happenTime"
            @change="updateHappenTime"
            ok-label="Confirmar" 
            clear-label="Limpar" 
            cancel-label="Cancelar" 
            :month-names="momentDate.months()"
            :day-names="momentDate.weekdays()" 
          />
        </q-field>

        <div class="row items-center">
          <!-- Duration -->
          <div class="col-8">
            <q-field
              :error="$v.task.durationTime.$error"
              error-label="* campo obrigatório!"
            >
              <q-input
                type="number"
                :min="0"
                :step="5"
                disable
                float-label="Duração da tarefa (min)"
                v-model="task.durationTime"
                @blur="$v.task.durationTime.$touch"
              />          
            </q-field>
          </div>
          <div class="col-4">
            <q-btn @click="task.durationTime += 5">+</q-btn>
            <q-btn @click="task.durationTime > 0 ? task.durationTime -= 5 : task.durationTime = 0">-</q-btn>
          </div>
        </div>

        <div class="row items-center">
          <!-- Lembrete -->
          <div class="col-8">
            <q-field
              :error="$v.task.reminder.$error"
              error-label="* campo obrigatório!"
            >
              <q-input
                type="number"
                :min="0"
                :step="5"
                disable
                float-label="Lembrete (min)"
                v-model="task.reminder"
                @blur="$v.task.reminder.$touch"
              />
            </q-field>
          </div>
          <div class="col-4">
            <q-btn @click="task.reminder += 5">+</q-btn>
            <q-btn @click="task.reminder > 0 ? task.reminder -= 5 : task.reminder = 0">-</q-btn>
          </div>
        </div>

        <q-field
        >
          <q-select
            multiple
            chips
            v-model="tags"
            :options="tagsOptions"
          />
        </q-field>

      </div>

    </q-modal-layout>
  </q-modal>
</template>

<script>
import {
  QModal,
  QModalLayout,
  QToolbar,
  QBtn,
  QField,
  QInput,
  QDatetime,
  QIcon,
  QSelect,
  Toast
} from 'quasar'
import {
  required
} from 'vuelidate/lib/validators'
import '../../node_modules/gun/lib/path'
import moment from 'moment'
import '../../node_modules/moment/locale/pt-br'
import _ from 'Lodash'
import guid from '../guid'

export default {
  components: {
    QModal,
    QModalLayout,
    QToolbar,
    QBtn,
    QField,
    QInput,
    QDatetime,
    QIcon,
    QSelect
  },
  data () {
    return {
      happenTime: new Date(),
      tags: [],
      tagsOptions: [],
      momentDate: moment,
      task: {
        id: undefined,
        description: undefined,
        happenTime: undefined,
        durationTime: 0,
        reminder: 0,
        creationDate: null,
        updateDate: null,
        deleteDate: null,
        done: false,
        group: undefined,
        tags: {}
      }
    }
  },
  validations: {
    task: {
      description: {
        required
      },
      happenTime: {
        required
      },
      durationTime: {
        required
      },
      reminder: {
        required
      }
    }
  },
  mounted () {
    // Busca a lista de tags disponíveis para seleção
    this.getTagOptions()
    // Configura o moment para a linguagem em pt-BR
    this.momentDate.locale('pt-br')
  },
  methods: {
    open (id) {
      let that = this

      this.happenTime = new Date()
      this.tags = []
      this.task = {
        id: undefined,
        description: undefined,
        happenTime: moment(this.happenTime).format('x'),
        durationTime: 0,
        reminder: 0,
        creationDate: null,
        updateDate: null,
        deleteDate: null,
        done: false,
        group: undefined,
        tags: {}
      }

      if (id) {
        this.$gun.get('task/' + id, function (ack) {
          if (ack.put) {
            that.happenTime = moment(ack.put.happenTime, 'x').toDate()
            that.task = _.extend(that.task, ack.put)
          }
        })

        delete this.task['tagsDescriptions']

        this.$gun.get('task/' + id).map().val((tsk, tid) => {
          if (tid === 'tags') {
            delete tsk['_']
            _.each(tsk, (tagID) => {
              that.tags.push(tagID)
            })
          }
        })
      }
      this.$refs.taskManagerModal.open()
    },
    updateHappenTime (value) {
      // Unix ms timestamp
      this.task.happenTime = value ? moment(value).format('x') : undefined
      this.$v.task.happenTime.$touch()
    },
    saveTask () {
      this.task.happenTime = this.happenTime ? moment(this.happenTime).format('x') : undefined
      let that = this
      this.$v.$touch()

      if (this.$v.$error) {
        Toast.create.negative('Preencha os campos corretamente!')
      }
      else {
        // seta o ID e os valores da data de criação e atualização
        if (!this.task.id) {
          this.task.creationDate = moment().format('x') // Unix ms timestamp
          this.task.updateDate = moment().format('x') // Unix ms timestamp
          this.task.id = guid()
        }

        this.$gun.get('task/' + this.task.id).path('tags').put(null)
        this.task.group = moment(this.task.happenTime, 'x').format('DD/MM/YYYY') // agrupamento será realizado por data                

        // Seta as tags selecionadas na tarefa
        this.task.tags = {}
        _.each(this.tags, (tag) => {
          this.task.tags[tag] = tag
        })

        // Cria ou atualiza uma tarefa
        let task = this.$gun.get('task/' + this.task.id).put(this.task, function (ack) {
          if (ack.err) {
            Toast.create.negative('Não foi possível gravar a tarefa no banco de dados! Favor tentar novamente!')
            console.log(ack.err)
            return
          }
          Toast.create.positive('Tarefa cadastrada com sucesso')
          that.$nextTick(() => {
            that.$refs.taskManagerModal.close()
          })
        })

        // Atualiza a lista de tasks
        this.$gun.get('tasks').set(task)

        this.$emit('updateListTask')
      }
    },
    getTagOptions () {
      let that = this
      this.$gun.get('tags').map().val(function (tag, id) {
        if (!tag.deleteDate) {
          that.tagsOptions.push({
            label: tag.description,
            value: tag.id
          })
        }
      })
    }
  }
}
</script>

<style>
</style>
