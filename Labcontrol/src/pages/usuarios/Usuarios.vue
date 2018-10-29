<template>
  <a-row>

    <!-- Cabeçalho da página -->
    <a-row style = "margin-bottom: 30px;">
      <!-- Por que os valores 16 e 4 para span e offset? -->
      <a-col :span = "16" :offset = "4" style = "text-align: center">
        <h1> Usuários </h1>
      </a-col>
    </a-row>

    <a-table :dataSource = "usuarios" :columns = "columns" :locale = "{ filterConfirm: 'Ok', filterReset: 'Resetar', emptyText: 'Nenhum Usuário Cadastrado' }">
      <span slot = "expandedRowRender" slot-scope = "record" style = "margin: 0">
        <p> <b> E-mail: </b> {{ record.email }} </p>
        <p> <b> Tipo de Usuário: </b> {{ record.type }} </p>
      </span>

      <span slot = "actions" slot-scope = "text">
        <a-tooltip placement = "top">
          <template slot = "title">
            <span> Deletar Usuário </span>
          </template>

          <a-tag @click = "showConfirmModal(text)" color = "red" :key = "text" >
            <a-icon type = "delete" />
          </a-tag>
        </a-tooltip>
      </span>
      
      <a-icon slot = "filterIcon" slot-scope = "filtered" type='search' :style = "{ color: filtered ? '#108ee9' : '#aaa' }" />
      
      <div slot = "filterDropdownRA" slot-scope = "{ setSelectedKeys, selectedKeys, confirm, clearFilters }" class = 'custom-filter-dropdown'>
        <a-input
          ref = "raInput"
          placeholder = 'Buscar por RA...'
          :value = "selectedKeys[0]"
          @change = "e => setSelectedKeys(e.target.value ? [e.target.value] : [])"
          @pressEnter = "() => handleSearch('searchRA', selectedKeys, confirm)"
        />
        <a-button type = 'primary' @click = "() => handleSearch('searchRA', selectedKeys, confirm)"> Buscar </a-button>
        <a-button @click = "() => handleReset('searchRA', clearFilters)"> Resetar </a-button>
      </div>

      <div slot = "filterDropdownNome" slot-scope = "{ setSelectedKeys, selectedKeys, confirm, clearFilters }" class = 'custom-filter-dropdown'>
        <a-input
          ref = "nomeInput"
          placeholder = 'Buscar nome...'
          :value = "selectedKeys[0]"
          @change = "e => setSelectedKeys(e.target.value ? [e.target.value] : [])"
          @pressEnter = "() => handleSearch('searchNome', selectedKeys, confirm)"
        />
        <a-button type = 'primary' @click = "() => handleSearch('searchNome', selectedKeys, confirm)"> Buscar </a-button>
        <a-button @click = "() => handleReset('searchNome', clearFilters)"> Resetar </a-button>
      </div>

      <div slot = "filterDropdownSobrenome" slot-scope = "{ setSelectedKeys, selectedKeys, confirm, clearFilters }" class = 'custom-filter-dropdown'>
        <a-input
          ref = "sobrenomeInput"
          placeholder = 'Buscar sobrenome...'
          :value = "selectedKeys[0]"
          @change = "e => setSelectedKeys(e.target.value ? [e.target.value] : [])"
          @pressEnter = "() => handleSearch('searchSobrenome', selectedKeys, confirm)"
        />
        <a-button type = 'primary' @click = "() => handleSearch('searchSobrenome', selectedKeys, confirm)"> Buscar </a-button>
        <a-button @click = "() => handleReset('searchSobrenome', clearFilters)"> Resetar </a-button>
      </div>

      <div slot = "filterDropdownCurso" slot-scope = "{ setSelectedKeys, selectedKeys, confirm, clearFilters }" class = 'custom-filter-dropdown'>
        <a-input
          ref = "cursoInput"
          :value = "selectedKeys[0]"
          @change = "e => setSelectedKeys(e.target.value ? [e.target.value] : [])"
          @pressEnter = "() => handleSearch('searchCurso', selectedKeys, confirm)"
        />
        <a-button type = 'primary' @click = "() => handleSearch('searchCurso', selectedKeys, confirm)"> Buscar </a-button>
        <a-button @click = "() => handleReset('searchCurso', clearFilters)"> Resetar </a-button>
      </div>

    </a-table>

    <a-modal
      :visible = "visibleConfirmModal"
      :footer = "null"
      @cancel = "closeConfirmModal()"
      style = "padding: 32px 32px 24px;"
    >
      <a-icon type = "question-circle-o" style = "color: #faad14; font-size: 22px; margin-right: 16px" />
      <span> <b> Cuidado! </b> </span> <br />
      <span style = "margin-left: 38px;"> Realmente deseja deletar o usuário: {{usuario}}? </span> <br />
      <span style = "margin-left: 38px;"> <i> Esta ação não poderá ser desfeita. </i> </span> <br/>

      <div style = "text-align: right; margin-top: 20px;">
        <a-button @click = "closeConfirmModal()"> Cancelar </a-button>
        <a-button @click = "deletaUsuario()" type = "danger"> Deletar </a-button>
      </div> 
    </a-modal>


  </a-row>
</template>

<script>
  import firebaseApp from '../../firebase-controller.js'

  const db = firebaseApp.database()
  const auth = firebaseApp.auth()

  export default {
    data () {
      return {
        role: null,
        usuarios: [],
        usuario: '',
        searchRA: '',
        columns: [{
          title: 'RA',
          dataIndex: 'ra',
          key: 'ra',
          scopedSlots: {
            filterDropdown: 'filterDropdownRA',
            filterIcon: 'filterIcon'
          },
          onFilter: (value, record) => record.ra.toLowerCase().includes(value.toLowerCase()),
          onFilterDropdownVisibleChange: (visible) => {
            if (visible) {
              setTimeout(() => {
                this.$refs.raInput.focus()
              })
            }
          }
        }, {
          title: 'Nome',
          dataIndex: 'nome',
          key: 'nome',
          scopedSlots: {
            filterDropdown: 'filterDropdownNome',
            filterIcon: 'filterIcon'
          },
          onFilter: (value, record) => record.nome.toLowerCase().includes(value.toLowerCase()),
          onFilterDropdownVisibleChange: (visible) => {
            if (visible) {
              setTimeout(() => {
                this.$refs.nomeInput.focus()
              })
            }
          }
        }, {
          title: 'Sobrenome',
          dataIndex: 'sobrenome',
          key: 'sobrenome',
          scopedSlots: {
            filterDropdown: 'filterDropdownSobrenome',
            filterIcon: 'filterIcon'
          },
          onFilter: (value, record) => record.sobrenome.toLowerCase().includes(value.toLowerCase()),
          onFilterDropdownVisibleChange: (visible) => {
            if (visible) {
              setTimeout(() => {
                this.$refs.sobrenomeInput.focus()
              })
            }
          }
        }, {
          title: 'Curso',
          dataIndex: 'curso',
          key: 'curso',
          filters: this.populaFiltroCursos(),
          onFilter: (value, record) => record.curso === value
        }, {
          title: 'Ações',
          dataIndex: 'id',
          key: 'acoes',
          align: 'center',
          scopedSlots: { customRender: 'actions' }
        }],
        visibleConfirmModal: false
      }
    },
    beforeMount: function () {
      let _this = this

      db.ref('Usuarios').orderByKey().on('value', function (snapshot) {
        _this.usuarios = []

        snapshot.forEach(function (item) {
          _this.usuarios.push({
            'id': item.key,
            'ra': item.val().RA,
            'email': item.val().Email,
            'type': item.val().role,
            'nome': item.val().Nome,
            'sobrenome': item.val().Sobrenome,
            'curso': item.val().Curso
          })
        })
      })

      db.ref('Usuarios/' + auth.currentUser.uid + '/role').on('value', function (snapshot) {
        _this.role = snapshot.val()
      })
    },
    methods: {
      handleSearch (inputText, selectedKeys, confirm) {
        confirm()
        this[inputText] = selectedKeys[0]
      },
      handleReset (inputText, clearFilters) {
        clearFilters()
        this[inputText] = ''
      },
      showConfirmModal (usuario) {
        this.usuario = usuario
        this.visibleConfirmModal = true
      },
      closeConfirmModal () {
        this.visibleConfirmModal = false
        this.usuario = ''
      },
      populaFiltroCursos () {
        var cursos = []
        db.ref('Controle/Cursos').orderByKey().on('value', function (snapshot) {
          snapshot.forEach(function (item) {
            cursos.push({
              'text': item.key,
              'value': item.key
            })
          })
        })

        return cursos
      },
      deletaUsuario () {
        let _this = this
        _this.visibleConfirmModal = false

        console.log(_this.usuario.id)
        db.ref('Usuarios').child(_this.usuario).remove().then(function () {
          _this.$notification.success({
            message: 'Yey!',
            description: 'Usuário deletado com sucesso.'
          })
        }).catch(() => {
          _this.$notification.error({
            message: 'Yey!',
            description: 'Falha ao deletar Usuário ' + _this.usuario
          })
        })

        _this.usuario = ''
      }
    }

  }
</script>

<style>
  .custom-filter-dropdown {
    padding: 8px;
    border-radius: 6px;
    background: #fff;
    box-shadow: 0 1px 6px rgba(0, 0, 0, .2);
  }

  .custom-filter-dropdown input {
    width: 130px;
    margin-right: 8px;
  }

  .custom-filter-dropdown button {
    margin-right: 8px;
  }

  .highlight {
    color: #f50;
  }
</style>
