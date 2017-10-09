<template>
  <!-- Modal para configurar o parâmetro  -->
  <q-modal ref="setParameterModal" noBackdropDismiss noEscDismiss :content-css="{minWidth: '20vw', minHeight: '280px'}">
    <q-modal-layout>
      <q-toolbar slot="header">
        <div class="q-toolbar-title">
          Configurar Parâmetro
        </div>
      </q-toolbar>
      <q-toolbar color="white" slot="footer" class="justify-end">
        <q-btn class="bg-primary" @click="setParameter" >Confirmar</q-btn>
      </q-toolbar>
      <div class="layout-padding">
        <q-option-group
          type="radio"
          v-model="parameter"
          :options="[
            { label: 'Dia', value: 'day' },
            { label: 'Semana', value: 'week' },
            { label: 'Mês', value: 'month' }
          ]"
          @blur="onBlur"
        />        
      </div>
    </q-modal-layout>
  </q-modal>
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
  QModal,
  QModalLayout,
  QOptionGroup
} from 'quasar'

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
    QModal,
    QModalLayout,
    QOptionGroup
  },
  props: {
    value: {
      type: String,
      required: true
    }
  },
  data () {
    return {
      parameter: this.value
    }
  },
  methods: {
    onBlur () {
      this.$emit('input', this.parameter)
    },
    setParameter () {
      let that = this
      // Grava o valor do parametro no banco de dados
      this.$gun.get('parameter').put({ value: that.parameter }, function (ack) {
        if (ack.err) {
          Toast.create.negative('Não foi possível gravar o parâmetro no banco de dados! Favor tentar novamente!')
          console.log(ack.err)
          return
        }
        Toast.create.positive('Parâmetro configurado com sucesso')
        that.$refs.setParameterModal.close()
      })
    },
    open () {
      this.$refs.setParameterModal.open()
    }
  }
}
</script>

<style>
</style>
