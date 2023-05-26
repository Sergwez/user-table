<template>
  <div class="card my-2">
    <h2 class="card-title my-4">Таблица пользователей</h2>
    <div class="card-body p-1">
      <table class="table table-hover table-bordered">
        <thead>
          <tr>
            <th scope="col">Имя</th>
            <th scope="col">Фамилия</th>
            <th scope="col">День рождения</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="user in users" :key="user.id">
            <td>{{ user.firstName }}</td>
            <td>{{ user.lastName }}</td>
            <td>{{ user.birthDay }}</td>
            <td @click="setCurrentUser(user)">
              <a
                href=""
                data-bs-toggle="modal"
                data-bs-target="#editModal"
                @click="isUpdate = true"
                >Изменить</a
              >
              <br />
              <a href="" data-bs-toggle="modal" data-bs-target="#deleteModal"
                >Удалить</a
              >
            </td>
          </tr>
        </tbody>
      </table>
      <button
        class="btn btn-primary d-inline"
        data-bs-toggle="modal"
        data-bs-target="#editModal"
        @click="clearCurrentUser()"
      >
        Добавить пользователя
      </button>
    </div>
    <nav class="d-block">
      <ul class="pagination justify-content-center">
        <li
          class="page-item"
          :class="1 == currentPage ? 'disabled' : ''"
          @click="changePage(1)"
        >
          <a class="page-link" href="#">В начало</a>
        </li>
        <li
          class="page-item"
          :class="page == currentPage ? 'active' : ''"
          v-for="page in pagination"
          :key="page"
          v-show="lastPage >= page"
        >
          <a
            class="page-link"
            :disabled="page === currentPage"
            href="#"
            @click="changePage(page)"
          >
            {{ page }}
          </a>
        </li>
        <li
          class="page-item"
          :class="lastPage == currentPage ? 'disabled' : ''"
          @click="changePage(lastPage)"
        >
          <a class="page-link" href="#">В конец</a>
        </li>
      </ul>
    </nav>
    <div
      class="modal fade"
      id="editModal"
      tabindex="-1"
      role="dialog"
      aria-labelledby="exampleModalLabel"
      aria-hidden="true"
    >
      <div
        class="modal-dialog"
        role="document"
        data-bs-config="{backdrop:true}"
      >
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="exampleModalLabel">
              Редактировать пользователя
            </h5>
            <button
              type="button"
              class="close btn"
              data-bs-dismiss="modal"
              aria-label="Close"
            >
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <div class="mb-3">
              <label for="exampleInputEmail1" class="form-label">Имя</label>
              <input
                placeholder="Иван"
                v-model="currentUser.firstName"
                type="text"
                class="form-control"
                id="firstName"
                aria-describedby="emailHelp"
              />
            </div>
            <div class="mb-3">
              <label for="exampleInputPassword1" class="form-label"
                >Фамилия</label
              >
              <input
                placeholder="Иванов"
                v-model="currentUser.lastName"
                type="text"
                class="form-control"
                id="lastName"
                required
              />
            </div>
            <div class="mb-3">
              <label for="exampleInputPassword1" class="form-label"
                >Дата рождения</label
              >
              <input
                placeholder="01.01.1900"
                v-model="currentUser.birthDay"
                type="date"
                class="form-control"
                id="birthDay"
                max="2010-01-01"
                required
              />
            </div>
          </div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-primary"
              @click="isUpdate ? updateUser() : addUser()"
              :disabled="!currentUser.lastName || !currentUser.birthDay"
            >
              Изменить
            </button>
          </div>
        </div>
      </div>
    </div>
    <div
      class="modal fade"
      id="deleteModal"
      tabindex="-1"
      role="dialog"
      aria-labelledby="deleteModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="deleteModalLabel">
              Удалить пользователя?
            </h5>
            <button
              type="button"
              class="close btn"
              data-bs-dismiss="modal"
              aria-label="Close"
            >
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-footer d-flex justify-content-between">
            <button
              type="button"
              class="btn btn-danger"
              @click="deleteUser(currentUser)"
            >
              Да, удалить
            </button>
            <button
              type="button"
              class="btn btn-primary"
              data-bs-dismiss="modal"
              aria-label="Close"
            >
              Отмена
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { Modal } from "bootstrap";

export default {
  data() {
    return {
      currentUser: {
        id: "",
        firstName: "",
        lastName: "",
        birthDay: "",
      },
      users: {},
      totalItems: "",
      limitItems: 3,
      currentPage: 1,
      isUpdate: false,
    };
  },
  computed: {
    pagination() {
      if (this.currentPage === 1) {
        return [1, 2, 3];
      }
      if (this.currentPage === this.lastPage && this.currentPage > 2) {
        return [this.currentPage - 2, this.currentPage - 1, this.currentPage];
      }
      return [this.currentPage - 1, this.currentPage, this.currentPage + 1];
    },
    lastPage() {
      return Math.ceil(this.totalItems / this.limitItems);
    },
  },
  methods: {
    async getUsers() {
      fetch(
        `http://localhost:3000/users/?_page=${this.currentPage}&_limit=${this.limitItems}`,
        {}
      ).then((res) => {
        if (res.status !== 200) {
          return console.error(res.statusText);
        }
        this.totalItems = res.headers.get("X-Total-Count");
        res.json().then((data) => {
          this.users = data;
        });
      });
    },
    async addUser() {
      fetch("http://localhost:3000/users/", {
        method: "POST",
        headers: {
          "Content-Type": "application/json;charset=utf-8",
        },
        body: JSON.stringify(this.currentUser),
      }).then((res) => {
        if (res.status === 201) {
          this.currentPage = 1;
          this.getUsers();
          this.closeModal("editModal");
          return;
        }
        return console.error(res.statusText);
      });
    },
    async updateUser() {
      fetch(`http://localhost:3000/users/${this.currentUser.id}`, {
        method: "PATCH",
        headers: {
          "Content-Type": "application/json;charset=utf-8",
        },
        body: JSON.stringify(this.currentUser),
      }).then((res) => {
        if (res.status === 200) {
          this.currentPage = 1;
          this.getUsers();
          this.closeModal("editModal");
          return;
        }
        return console.error(res.statusText);
      });
    },
    async deleteUser(user) {
      fetch(`http://localhost:3000/users/${user.id}`, {
        method: "DELETE",
      }).then((res) => {
        if (res.status === 200) {
          this.currentPage = 1;
          this.getUsers();
          this.closeModal("deleteModal");
          return;
        }
        return console.error(res.statusText);
      });
    },
    closeModal(idModal) {
      Modal.getOrCreateInstance(document.getElementById(idModal)).hide();
    },
    changePage(page) {
      this.currentPage = page;
      this.getUsers();
    },
    setCurrentUser(user) {
      this.currentUser = Object.assign({}, user);
    },
    clearCurrentUser() {
      this.isUpdate = false;
      Object.keys(this.currentUser).forEach((elem) => {
        this.currentUser[elem] = "";
      });
    },
  },
  mounted() {
    this.getUsers();
  },
};
</script>

<style scoped>
.table {
  @media (max-width: 480px) {
    font-size: 0.75rem;
  }
}
</style>
