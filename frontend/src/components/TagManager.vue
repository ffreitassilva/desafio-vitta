<template>
  <q-modal ref="tagModal" :content-css="{minWidth: '30vw', minHeight: '300px'}">
    <q-modal-layout>

      <q-toolbar slot="header">
        <div class="q-toolbar-title">
          Tag
        </div>
        <q-btn flat @click="$refs.tagModal.close()">
          <q-icon name="close" />
        </q-btn>
      </q-toolbar>

      <q-toolbar color="white" slot="footer" class="justify-end">
        <q-btn class="bg-primary" @click="saveTag">Confirmar</q-btn>
      </q-toolbar>

      <div class="layout-padding">
        <!-- Description -->
        <q-field
          :error="$v.tag.description.$error"
          error-label="* campo obrigatório!"
        >
          <q-input
            type="text"
            float-label="Descrição da tag"
            v-model="tag.description"
            @blur="$v.tag.description.$touch"
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
  Toast
} from 'quasar'
import {
  required
} from 'vuelidate/lib/validators'
import guid from '../guid'
import moment from 'moment'
import _ from 'Lodash'

export default {
  components: {
    QModal,
    QModalLayout,
    QToolbar,
    QBtn,
    QField,
    QInput,
    QDatetime,
    QIcon
  },
  data () {
    return {
      tag: {
        id: undefined,
        description: undefined,
        creationDate: null,
        updateDate: null,
        deleteDate: null
      }
    }
  },
  validations: {
    tag: {
      description: {
        required
      }
    }
  },
  methods: {
    open (id) {
      let that = this
      if (id) {
        this.$gun.get('tag/' + id, function (ack) {
          if (ack.put) {
            that.tag = _.extend(that.tag, ack.put)
          }
        })
      }
      else {
        this.tag = {
          id: undefined,
          description: undefined,
          creationDate: null,
          updateDate: null,
          deleteDate: null
        }
      }
      this.$refs.tagModal.open()
    },
    saveTag () {
      let that = this
      this.$v.$touch()

      if (this.$v.$error) {
        Toast.create('Preencha os campos corretamente!')
      }
      else {
        // seta o ID e os valores da data de criação e atualização
        if (!this.tag.id) {
          this.tag.creationDate = moment().format('x') // Unix ms timestamp
          this.tag.updateDate = moment().format('x') // Unix ms timestamp
          this.tag.id = guid()
        }

        // Cria ou atualiza uma tag
        let tag = this.$gun.get('tag/' + this.tag.id).put(this.tag, function (ack) {
          if (ack.err) {
            Toast.create.negative('Não foi possível gravar a tag no banco de dados! Favor tentar novamente!')
            console.log(ack.err)
            return
          }
          Toast.create.positive('Tag cadastrada com sucesso')
          that.$nextTick(() => {
            that.$refs.tagModal.close()
          })
        })

        // Atualiza a lista de tags
        this.$gun.get('tags').set(tag)
      }
    }
  }
}
</script>

<style>
</style>
