<template>
  <div class="container">
    <div class="row">
      <div class="col-md-8">
        <input type="text" v-model="search" class="form-control" placeholder="Пошук">
      </div>
      <div class="col-md-1">
        <button type="button" @click="showModal = true" class="btn btn-primary" data-toggle="modal" data-target="#exampleModalCenter">
          Додати
        </button>
      </div>
      <div class="col-md-2">
        <button class="btn btn-success" @click="fetchStadiums">
          Усі
        </button>
      </div>
    </div>
    <br>

    <div class="row">
      <div class="col-md-3">
        <b-form-input v-model="capacity" type="number" class="form-control"  step="1000"></b-form-input>
      </div>
      <div class="col-md-1">
        <button @click="getStadiumWithCapacityGreaterThan" class="btn btn-success">
          Пошук
        </button>
      </div>
      <div class="col-md-3">
        <button @click="getStadiumWithTreadmill" class="btn btn-success">
          З біговими доріжками
        </button>
      </div>
      <div class="col-md-2">
        <button @click="getStadiumOrderByCapacityDesc" class="btn btn-success">
          Сортувати
        </button>
      </div>
      <div class="col-md-2">
        <button @click="getStadiumOrderByCapacityAsc" class="btn btn-success">
          Сортувати
        </button>
      </div>
    </div>

    <b-card  v-for="stadium in filteredList" :key="stadium.id" v-bind:data="stadium" :title="stadium.name"
             img-src="../static/img/stadium.jpg"
             img-alt="Image"
             img-top
             tag="article"
             style="max-width: 20rem;"
             class="mb-2">
      <p class="card-text">
        <p>Дата заснування: {{stadium.foundationDate}}</p>
        <p>Адреса: {{stadium.address}}</p>
        <p>Вмістимість: {{stadium.capacity}}</p>
        <p>Бігові доріжки <font-awesome-icon v-if="stadium.treadmill" :icon="icon"/> <font-awesome-icon v-else :icon="iconFalse"/></p>
      </p>
      <b-button variant="success" @click="showUpdateModals(stadium)">Змінити</b-button>
      <b-button @click="deleteStadium(stadium.stadiumId)" variant="danger">Видалити</b-button>
    </b-card>

    <!-- Modal -->
    <div v-if="showModal">
      <transition name="modal">
        <div class="modal-mask">
          <div class="modal-wrapper">
            <div class="modal-dialog sss">
              <div class="modal-content">
                <div class="modal-header">
                  Додати
                  <button type="button" class="close" @click="showModal=false">
                    <span aria-hidden="true">&times;</span>
                  </button>

                </div>
                <div class="modal-body">

                  <div class="form-group">
                    <label>Назва</label>
                    <input v-model="formAdd.name" type="text" class="form-control" placeholder="Введіть назву" required>
                  </div>


                  <div class="form-group">
                    <label>Дата заснування</label>
                    <input v-model="formAdd.foundationDate" type="date" class="form-control" required>
                  </div>

                  <div class="form-group">
                    <label>Адреса</label>
                    <input v-model="formAdd.address" type="text" class="form-control" placeholder="Введіть адресу" required>
                  </div>

                  <div class="form-group">
                    <label>Вмістимість</label>
                    <b-form-input v-model="formAdd.capacity" type="number" class="form-control" placeholder="Введіть вмістимість" required></b-form-input>
                  </div>

                  <div class="form-group">
                    <label>Бігові доріжки &nbsp;&nbsp;</label>
                    <b-form-checkbox v-model="formAdd.treadmill"></b-form-checkbox>
                  </div>

                  <button class="btn btn-default" @click="showModal=false">Скасувати</button>
                  <button class="btn btn-success" @click="addStadium" >Зберегти</button>

                </div>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </div>

    <div v-if="showUpdateModal">
      <transition name="modal">
        <div class="modal-mask">
          <div class="modal-wrapper">
            <div class="modal-dialog sss">
              <div class="modal-content">
                <div class="modal-header">
                  Додати
                  <button type="button" class="close" @click="showUpdateModal=false">
                    <span aria-hidden="true">&times;</span>
                  </button>

                </div>
                <div class="modal-body">

                  <div class="form-group">
                    <label>Назва</label>
                    <input v-model="formAdd.name" type="text" class="form-control" placeholder="Введіть назву">
                  </div>


                  <div class="form-group">
                    <label>Дата заснування</label>
                    <input v-model="formAdd.foundationDate" type="date" class="form-control" >
                  </div>

                  <div class="form-group">
                    <label>Адреса</label>
                    <input v-model="formAdd.address" type="text" class="form-control" placeholder="Введіть адресу">
                  </div>

                  <div class="form-group">
                    <label>Вмістимість</label>
                    <input v-model="formAdd.capacity" type="number" class="form-control" placeholder="Введіть вмістимість">
                  </div>

                  <div class="form-group">
                    <label>Бігові доріжки &nbsp;&nbsp;</label>
                    <b-form-checkbox v-model="formAdd.treadmill"></b-form-checkbox>
                  </div>

                  <button class="btn btn-default" @click="showUpdateModal=false">Скасувати</button>
                  <button class="btn btn-success" @click="updateStadium(formAdd)">Зберегти</button>

                </div>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </div>



  </div>
</template>

<script>
  import img from '../../static/img/coach.jpg'
  import FontAwesomeIcon from '@fortawesome/vue-fontawesome'
  import faCheck from '@fortawesome/fontawesome-free-solid/faCheck'
  import faCross from '@fortawesome/fontawesome-free-solid/faTimes'
  import axios from 'axios'
    export default {
        name: "Stadium",
      data: () => ({
        search: '',
        stadiums: [],
        capacity: {},
        formAdd: {},
        showModal: false,
        showUpdateModal: false,
      }),
      created() {
        this.fetchStadiums();
      },
      methods: {
        fetchStadiums() {
          axios.get('http://localhost:8080/api/stadiums').then((response) => {
            this.stadiums = response.data;
          });
        },
        getStadiumWithCapacityGreaterThan() {
          axios.get('http://localhost:8080/api/stadiums-greater/' + this.capacity).then((response) => {
            this.stadiums = response.data;
          })
        },
        getStadiumWithTreadmill() {
          axios.get('http://localhost:8080/api/stadiums-treadmill').then((response) => {
            this.stadiums = response.data;
          })
        },
        getStadiumOrderByCapacityDesc() {
          axios.get('http://localhost:8080/api/stadiums-order-desc').then((response) => {
            this.stadiums = response.data;
          })
        },
        getStadiumOrderByCapacityAsc() {
          axios.get('http://localhost:8080/api/stadiums-order-asc').then((response) => {
            this.stadiums = response.data;
          })
        },
        deleteStadium(id) {
          axios.delete('http://localhost:8080/api/stadiums/' + id).then(() => {
            this.fetchStadiums();
          });
        },
        addStadium() {
          this.showModal = false;
          axios.post('http://localhost:8080/api/stadiums/', this.formAdd).then(() => {
            this.fetchStadiums();
          });
          this.formAdd = {};
        },
        updateStadium(stadium) {
          this.showUpdateModal = false;
          axios.put('http://localhost:8080/api/stadiums/' + stadium.stadiumId, stadium).then(() => {
            this.fetchStadiums();
          });
          this.formAdd = {};
        },
        showUpdateModals(stadium) {
          this.showUpdateModal = true;
          this.formAdd = stadium;
        },
      },
      computed: {
        filteredList() {
          return this.stadiums.filter(stadium => {
            return stadium.name.toLowerCase().includes(this.search.toLowerCase());
          })
        },
        icon(){
          return faCheck;
        },
        iconFalse() {
          return faCross;
        }
      },
      components:{
          FontAwesomeIcon
      }
    }
</script>

<style scoped>
  .container {
    margin-top: 20px;
  }
  .card {
    display: inline-block;
    margin: 8px;
  }

  .modal-mask {
    position: fixed;
    z-index: 9998;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, .5);
    display: table;
    transition: opacity .3s ease;
    overflow-y: scroll;
  }

  .modal-wrapper {
    display: table-cell;
    vertical-align: middle;
  }
</style>
