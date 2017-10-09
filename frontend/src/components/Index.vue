<template>
  <q-layout 
    ref="layout" 
    view="hHh LpR lFf" 
    :right-class="{'bg-dark': true, 'text-white': true}" 
    :right-breakpoint="1100"
    :left-class="{'bg-dark': true, 'text-white': true}" 
  >

  <!-- Header -->
  <q-toolbar slot="header" color="dark">
    <q-btn flat @click="$refs.layout.toggleLeft()">
      <q-icon name="menu" />
    </q-btn>
    <q-toolbar-title>
      Desafio Vitta
      <span slot="subtitle">Task List</span>
    </q-toolbar-title>
    <q-btn flat @click="$refs.setParameterModal.open()">
      <q-icon name="settings" />
    </q-btn>
    <q-btn flat @click="$refs.layout.toggleRight()">
      <q-icon name="menu" />
    </q-btn>
  </q-toolbar>

  <!-- Filtros de tarefas -->
  <q-card class="filter-card">
    <q-card-main class="row items-center">
      <div class="col-sm-2">
        <q-datetime 
          v-model="baseDate" 
          format="DD/MM/YYYY" 
          type="date"
          ok-label="Confirmar" 
          clear-label="Limpar" 
          cancel-label="Cancelar" 
          @change="getTasks()"
          :month-names="momentDate.months()"
          :day-names="momentDate.weekdays()"
        />
      </div>
      <div class="col-sm-5 text-center">
        <q-btn 
          small 
          @click="getTasks();filterDone = 'all'" 
          :class="{'bg-teal-4': filterDone === 'all', 'text-white': filterDone === 'all'}"
        >
          Todas
        </q-btn>
        <q-btn 
          small 
          @click="getTasks();filterDone = 'done'" 
          :class="{'bg-teal-4': filterDone === 'done', 'text-white': filterDone === 'done'}"
        >
          Realizadas
        </q-btn>
        <q-btn 
          small 
          @click="getTasks();filterDone = 'notDone'" 
          :class="{'bg-teal-4': filterDone === 'notDone', 'text-white': filterDone === 'notDone'}"
        >
          Não Realizadas
        </q-btn>
      </div>
      <div class="col-sm-3 text-center">
        <q-btn 
          small 
          @click="getTasks();typeDisplay = 'day'" 
          :class="{'bg-teal-4': typeDisplay === 'day', 'text-white': typeDisplay === 'day'}"
        >
          Dia
        </q-btn>
        <q-btn 
          small 
          @click="getTasks();typeDisplay = 'week'" 
          :class="{'bg-teal-4': typeDisplay === 'week', 'text-white': typeDisplay === 'week'}"
        >
          Semana
        </q-btn>
        <q-btn 
          small 
          @click="getTasks();typeDisplay = 'month'" 
          :class="{'bg-teal-4': typeDisplay === 'month', 'text-white': typeDisplay === 'month'}"
        >
          Mês
        </q-btn>
      </div>
      <div class="col-sm-2 text-center">
        <q-btn small icon="create" @click="$refs.taskManagerModal.open()">Nova Tarefa</q-btn>
      </div>
      <div class="col-12">
        <q-search 
          placeholder="Buscar tarefa por descrição" 
          v-model="searchTask" 
          :debounce="300" 
          color="none" 
          @change="getTasks"
        />
      </div>
    </q-card-main>
  </q-card>

  <div class="row">
    <q-card 
      v-for="(tasks, date) in orderedGroupedTasks"
      inline 
      :key="date"
      :style="{width: '300px', height: '300px'}">
      <q-card-title class="text-center bg-primary text-white">
        <strong>{{date}}</strong>
      </q-card-title>
      <q-card-separator />
      <q-card-main>
        <div class="list-tasks">
          <q-list highlight multiline separator no-border>
            <q-item v-for="task in tasks" :key="task.id">
              <q-item-side>
                <q-checkbox v-model="task.done" @input="taskDone(task.id, task.done)"/>
              </q-item-side>
              <q-item-main :class="{done: task.done }">
                 <q-item-tile label>{{ task.description }}</q-item-tile>
                 <q-item-tile sublabel>{{ momentDate(task.happenTime, 'x').format('DD/MM/YYYY HH:mm') }}</q-item-tile>
                 <q-item-tile sublabel>
                   <q-chip small v-for="(tag, index) in task.tagsDescriptions" :key="index">
                    {{tag}}
                   </q-chip>
                 </q-item-tile>
              </q-item-main>
              <q-item-side class="row">
                <q-btn class="col-6 task-btn" small flat icon="create" @click="editTask(task.id)" v-if="!task.done"></q-btn>
                <q-btn class="col-6 task-btn" small flat icon="delete" @click="deleteTask(task.id)" v-if="!task.done"></q-btn>
              </q-item-side>
            </q-item>
            <q-item-separator />
          </q-list>
        </div>
      </q-card-main>
    </q-card>
  </div>  

  <!-- Componente para configurar o parâmetro de dia, semana ou mês -->
  <my-parameter v-model="parameter" ref="setParameterModal"></my-parameter>
  <!-- Componente criar ou editar uma tarefa -->
  <my-task-manager ref="taskManagerModal" @updateListTask="getTasks"></my-task-manager>
  <!-- Componente criar ou editar uma tag -->
  <my-tag-manager ref="tagManagerModal"></my-tag-manager>

  <!-- Right Side Panel -->
  <div slot="right">
    <q-toolbar-title class="text-center" :style="{ padding: '25px' }">
      Tag
    </q-toolbar-title>
    <div class="text-center">
      <q-btn small icon="create" @click="$refs.tagManagerModal.open()">Nova Tag</q-btn>
    </div>
    <q-search inverted placeholder="Buscar Tag" v-model="searchTag" color="none" @change="getTags"/>
    <q-list highlight multiline separator no-border>
      <q-item v-for="tag in tags" :key="tag.id">
        <q-item-main>
            <q-item-tile label>{{ tag.description }}</q-item-tile>
        </q-item-main>
        <q-item-side class="row">
          <q-btn class="col-6 task-btn text-white" small flat icon="create" @click="editTag(tag.id)"></q-btn>
          <q-btn class="col-6 task-btn text-white" small flat icon="delete" @click="deleteTag(tag.id)"></q-btn>
        </q-item-side>
      </q-item>
      <q-item-separator />
    </q-list>
  </div>
 
  <!-- Left Side Panel -->
  <div slot="left">
    <q-card>
      <q-card-title>
        Resolução de Tarefas
      </q-card-title>
      <q-card-separator />
      <q-card-main>
        <div class="echarts">
          <IEcharts :option="taskChart"></IEcharts>
        </div>
      </q-card-main>
    </q-card>
    <q-card>
      <q-card-title>
        Tag Resolvidas
      </q-card-title>
      <q-card-separator />
      <q-card-main>
        <div class="echarts">
          <IEcharts :option="tagChart"></IEcharts>
        </div>
      </q-card-main>
    </q-card>
  </div>
  
</q-layout>
</template>

<script>
import {
  QLayout,
  QToolbar,
  QToolbarTitle,
  QSearch,
  QTabs,
  QRouteTab,
  QBtn,
  QIcon,
  QItemSide,
  QItemMain,
  QSideLink,
  QListHeader,
  QScrollArea,
  QDatetime,
  QCard,
  QCardMain,
  Toast,
  QCardTitle,
  QCardSeparator,
  QList,
  QItem,
  QItemSeparator,
  QCheckbox,
  QItemTile,
  QChip
} from 'quasar'
import moment from 'moment'
import '../../node_modules/moment/locale/pt-br'
import _ from 'Lodash'
import '../../node_modules/gun/lib/path'
import myParameter from '../components/Parameter'
import myTaskManager from '../components/TaskManager'
import myTagManager from '../components/TagManager'
import IEcharts from 'vue-echarts-v3/src/full.vue'

export default {
  components: {
    QLayout,
    QToolbar,
    QToolbarTitle,
    QSearch,
    QTabs,
    QRouteTab,
    QBtn,
    QIcon,
    QItemSide,
    QItemMain,
    QSideLink,
    QListHeader,
    QScrollArea,
    QDatetime,
    QCard,
    QCardMain,
    QCardTitle,
    QCardSeparator,
    QList,
    QItem,
    QItemSeparator,
    QCheckbox,
    QItemTile,
    QChip,
    myParameter,
    myTaskManager,
    myTagManager,
    IEcharts
  },
  data () {
    return {
      baseDate: undefined,
      momentDate: moment,
      searchTag: '',
      searchTask: '',
      parameter: 'week',
      typeDisplay: undefined,
      tasks: [],
      groupedTask: [],
      tags: [],
      filterDone: 'all',
      doneTask: 0,
      notDoneTask: 0,
      doneTag: []
    }
  },
  computed: {
    orderedGroupedTasks () {
      return _.groupBy(_.orderBy(this.tasks, ['happenTime'], ['asc']), 'group')
    },
    tagChart () {
      console.log(_.map(_.groupBy(this.doneTag), (value, key) => { return key }))
      return {
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} ({d}%)',
          position: [10, 10]
        },
        legend: {
          show: true,
          orient: 'horizontal',
          bottom: 0,
          data: _.map(_.groupBy(this.doneTag), (value, key) => { return key }),
          textStyle: {
            color: '#fff'
          }
        },
        series: [
          {
            name: 'Tags',
            type: 'pie',
            radius: ['40%', '70%'],
            avoidLabelOverlap: false,
            label: {
              normal: {
                show: false,
                position: 'center'
              },
              emphasis: {
                show: true,
                textStyle: {
                  fontSize: '15',
                  fontWeight: 'bold',
                  color: '#fff'
                }
              }
            },
            labelLine: {
              normal: {
                show: false
              }
            },
            data: _.map(_.groupBy(this.doneTag), (value, key) => { return { value: value.length, name: key } })
          }
        ]
      }
    },
    taskChart () {
      return {
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} ({d}%)',
          position: [10, 10]
        },
        legend: {
          show: true,
          orient: 'horizontal',
          bottom: 0,
          data: ['Realizadas', 'Não Realizadas'],
          textStyle: {
            color: '#fff'
          }
        },
        series: [
          {
            name: 'Tarefas',
            type: 'pie',
            radius: ['40%', '70%'],
            avoidLabelOverlap: false,
            label: {
              normal: {
                show: false,
                position: 'center'
              },
              emphasis: {
                show: true,
                textStyle: {
                  fontSize: '15',
                  fontWeight: 'bold',
                  color: '#fff'
                }
              }
            },
            labelLine: {
              normal: {
                show: false
              }
            },
            data:
            [
              {value: this.doneTask, name: 'Realizadas'},
              {value: this.notDoneTask, name: 'Não Realizadas'}
            ]
          }
        ]
      }
    }
  },
  mounted () {
    this.configParameter()
    // Configura o moment para a linguagem em pt-BR
    this.momentDate.locale('pt-br')
    // Seta por default a data de hoje como data base
    this.baseDate = new Date()

    this.getTasks()
    this.getTags()
  },
  methods: {
    configParameter () {
      let that = this // instância do vue
      // Tenta encontrar no banco de dados se o parâmetro já foi configurado
      this.$gun.get('parameter', function (ack) {
        // Mostrar menssagem caso ocorra algum erro ao tentar buscar o parâmetro e salvar o valor default (semana)
        if (ack.err) {
          Toast.create.negative('Não foi possível obter o parâmetro do banco de dados!')
          console.log(ack.err)

          // abrir o modal para re-configurar o parâmetro
          that.$refs.setParameterModal.open()
        }
        // Se o parâmetro ainda não foi configurado, abrir o modal para configuração do mesmo
        else if (!ack.put) {
          that.$refs.setParameterModal.open()
        }
        else {
          that.typeDisplay = ack.put.value
        }
      })
    },
    getTasks () {
      let that = this
      this.tasks = []
      this.doneTag = []
      this.doneTask = 0
      this.notDoneTask = 0
      this.$gun.get('tasks').map().val(function (task, id) {
        // Filtragem de Tarefas
        let taskDate = that.momentDate(task.happenTime, 'x').toDate()
        if (!task.deleteDate && // não exibe as tarefas excluidas
            taskDate >= that.startDate() && // data da tarefa é maior ou igual a data de inicio
            taskDate <= that.endDate() && // data da tarefa é menor ou igual a data de fim
            (that.filterDone === 'all' || // exibe tarefas realizadas e não realizadas
            (that.filterDone === 'done' && task.done) || // somente as tarefas realizadas
            (that.filterDone === 'notDone' && !task.done)) && // somente as tarefas não realizadas
            (!that.searchTask || // nenhum termo de busca
             (that.searchTask &&
              (task.description.toUpperCase().includes(that.searchTask.toUpperCase()))))) { // filtra a descrição e tag pelo termo de busca
          delete task['_']
          let listTags = []
          // Busca as tags da task
          that.$gun.get('task/' + task.id).map().val((tsk, id) => {
            if (id === 'tags') {
              delete tsk['_']
              _.each(tsk, (tagID) => {
                that.$gun.get('tag/' + tagID).val((tag) => {
                  listTags.push(tag.description)
                  if (task.done) {
                    that.doneTag.push(tag.description)
                  }
                })
              })
            }
          })
          task = _.extend(task, { tagsDescriptions: listTags })
          that.tasks.push(task)
          if (task.done) {
            that.doneTask += 1
          }
          else {
            that.notDoneTask += 1
          }
        }
      })
    },
    getTags () {
      let that = this
      this.tags = []
      this.$gun.get('tags').map().val(function (tag, id) {
        // Filtragem de tags
        if (!tag.deleteDate && // não exibe as tags excluidas
            (!that.searchTag || // nenhum termo de busca
             (that.searchTag &&
              tag.description.toUpperCase().includes(that.searchTag.toUpperCase())))) { // filtra a descrição pelo termo de busca
          delete tag['_']
          that.tags.push(tag)
        }
      })
    },
    taskDone (id, done) {
      this.getTasks()
      this.$gun.get('task/' + id).path('done').put(done)
    },
    editTask (id) {
      this.$refs.taskManagerModal.open(id)
    },
    deleteTask (id) {
      this.$gun.get('task/' + id).path('deleteDate').put(moment().format('x'))
      this.getTasks()
    },
    editTag (id) {
      this.$refs.tagManagerModal.open(id)
    },
    deleteTag (id) {
      this.$gun.get('tag/' + id).path('deleteDate').put(moment().format('x'))
      this.getTags()
    },
    startDate () {
      return this.typeDisplay === 'day'
        ? this.momentDate(this.baseDate).startOf('day').toDate()
        : this.typeDisplay === 'week'
          ? this.momentDate(this.baseDate).startOf('week').toDate()
          : this.momentDate(this.baseDate).startOf('month').toDate()
    },
    endDate () {
      return this.typeDisplay === 'day'
        ? this.momentDate(this.baseDate).endOf('day').toDate()
        : this.typeDisplay === 'week'
          ? this.momentDate(this.baseDate).endOf('week').toDate()
          : this.momentDate(this.baseDate).endOf('month').toDate()
    }
  }
}
</script>

<style scoped lang="stylus">
.filter-card
  box-shadow none !important

.list-tasks
  overflow auto
  height 200px

.done
  text-decoration line-through
  color #c1c1c1

.task-btn
  padding 0

.q-chip
  &.small 
    font-size 10px !important

.echarts
  width 250px
  height 250px
</style>
