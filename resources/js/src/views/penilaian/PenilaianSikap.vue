<template>
  <b-card no-body>
    <b-card-body>
      <div v-if="isBusy" class="text-center text-danger my-2">
        <b-spinner class="align-middle"></b-spinner>
        <strong>Loading...</strong>
      </div>
      <div v-else>
        <template v-if="kurtilas">
          <datatable :loading="loading" :isBusy="isBusy" :items="items" :fields="fields" :meta="meta" @per_page="handlePerPage" @pagination="handlePagination" @search="handleSearch" @aksi="handleAksi" />
        </template>
        <template v-else>
          <b-alert show variant="danger">
            <div class="alert-body">
              <p>Penilaian Sikap hanya bagi Satuan Pendidikan yang memiliki Rombongan Belajar Kurikulum 2013!</p>
            </div>
          </b-alert>
        </template>
      </div>
    </b-card-body>
  </b-card>
</template>

<script>
import { BCard, BCardBody, BSpinner, BAlert } from 'bootstrap-vue'
import Datatable from './Datatable.vue'
import eventBus from '@core/utils/eventBus';
export default {
  components: {
    BCard,
    BCardBody,
    BSpinner,
    BAlert,
    Datatable
  },
  data() {
    return {
      loading: false,
      isBusy: true,
      fields: [
        {
          key: 'nama_pd',
          label: 'Nama PD',
          sortable: false,
          thClass: 'text-center',
        },
        {
          key: 'rombel',
          label: 'Kelas',
          sortable: false,
          thClass: 'text-center',
        },
        {
          key: 'dimensi_sikap',
          label: 'Dimensi Sikap',
          sortable: false,
          thClass: 'text-center',
        },
        {
          key: 'elemen_sikap',
          label: 'Elemen Sikap',
          sortable: false,
          thClass: 'text-center',
          tdClass: 'text-center'
        },
        {
          key: 'opsi_sikap',
          label: 'Opsi Sikap',
          sortable: false,
          thClass: 'text-center',
          tdClass: 'text-center'
        },
        {
          key: 'deskripsi',
          label: 'Uraian Sikap',
          sortable: false,
          thClass: 'text-center',
        },
        {
          key: 'actions',
          label: 'Aksi',
          sortable: false,
          thClass: 'text-center',
          tdClass: 'text-center'
        },
      ],
      items: [],
      meta: {},
      current_page: 1, //DEFAULT PAGE YANG AKTIF ADA PAGE 1
      per_page: 10, //DEFAULT LOAD PERPAGE ADALAH 10
      search: '',
      sortBy: 'created_at', //DEFAULT SORTNYA ADALAH CREATED_AT
      sortByDesc: true, //ASCEDING
      kurtilas: null,
    }
  },
  created() {
    eventBus.$on('add-sikap', (val) => {
      if(!this.kurtilas){
        this.$swal({
          icon: 'error',
          title: 'GAGAL',
          text: 'Penilaian Sikap hanya bagi Satuan Pendidikan yang memiliki Rombel Kurtilas',
          customClass: {
            confirmButton: 'btn btn-success',
          },
        })
      } else {
        this.$router.push({ name: 'penilaian-input-sikap' })
      }
    })
    this.loadPostsData()
  },
  methods: {
    loadPostsData() {
      this.loading = true
      //let current_page = this.search == '' ? this.current_page : this.current_page != 1 ? 1 : this.current_page
      let current_page = this.current_page//this.search == '' ? this.current_page : 1
      //LAKUKAN REQUEST KE API UNTUK MENGAMBIL DATA POSTINGAN
      this.$http.get('/penilaian/nilai-sikap', {
        params: {
          user_id: this.user.user_id,
          guru_id: this.user.guru_id,
          sekolah_id: this.user.sekolah_id,
          semester_id: this.user.semester.semester_id,
          periode_aktif: this.user.semester.nama,
          page: current_page,
          per_page: this.per_page,
          q: this.search,
          sortby: this.sortBy,
          sortbydesc: this.sortByDesc ? 'DESC' : 'ASC'
        }
      }).then(response => {
        //this.items = response.data.all_pd
        let getData = response.data.data
        this.isBusy = false
        this.loading = false
        this.items = getData.data//MAKA ASSIGN DATA POSTINGAN KE DALAM VARIABLE ITEMS
        //DAN ASSIGN INFORMASI LAINNYA KE DALAM VARIABLE META
        this.kurtilas = response.data.kurtilas
        this.meta = {
          total: getData.total,
          current_page: getData.current_page,
          per_page: getData.per_page,
          from: getData.from,
          to: getData.to,
          role_id: this.role_id,
          roles: response.data.roles,
        }
      })
    },
    //JIKA ADA EMIT TERKAIT LOAD PERPAGE, MAKA FUNGSI INI AKAN DIJALANKAN
    handlePerPage(val) {
      this.per_page = val //SET PER_PAGE DENGAN VALUE YANG DIKIRIM DARI EMIT
      this.loadPostsData() //DAN REQUEST DATA BARU KE SERVER
    },
    handleRole(val){
      this.role_id = val
      this.loadPostsData()
    },
    //JIKA ADA EMIT PAGINATION YANG DIKIRIM, MAKA FUNGSI INI AKAN DIEKSEKUSI
    handlePagination(val) {
      this.current_page = val //SET CURRENT PAGE YANG AKTIF
      this.loadPostsData()
    },
    //JIKA ADA DATA PENCARIAN
    handleSearch(val) {
      this.search = val //SET VALUE PENCARIAN KE VARIABLE SEARCH
      this.loadPostsData() //REQUEST DATA BARU
    },
    handleAksi(val){
      console.log(val);
    },
  },
}
</script>
<style lang="scss">
@import '~@resources/scss/vue/libs/vue-sweetalert.scss';
</style>