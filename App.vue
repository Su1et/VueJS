<template>
  <h1>Створіть акаунт</h1>
  <div>
    <form @submit.prevent="onSubmit" novalidate>
      <fieldset>
        <label for="first_name">
          Ім'я:
          <input
              id="first_name"
              type="text"
              v-model="user.firstName"
              required
              @keyup="v$.firstName.$touch()"
          />
          <span v-if="v$.firstName.$error" class="error">Введіть коректне ім'я</span>
        </label>

        <label for="last_name">
          Прізвище:
          <input
              id="last_name"
              type="text"
              v-model="user.lastName"
              required
              @keyup="v$.lastName.$touch()"
          />
          <span v-if="v$.lastName.$error" class="error">Введіть коректне прізвище</span>
        </label>

        <label for="email">
          E-mail:
          <input
              id="email"
              type="email"
              v-model="user.email"
              required
              @keyup="v$.email.$touch()"
          />
          <span v-if="v$.email.$error" class="error">Введіть коректну електронну адресу</span>
        </label>

        <label for="phone">
          Номер телефону:
          <input
              id="phone"
              type="tel"
              v-model="user.phone"
              v-imask="phoneNumberMask"
              placeholder="+38(___)-___-__-__"
              required
              @keyup="v$.phone.$touch()"
          />
          <span v-if="v$.phone.$error" class="error">Введіть коректний номер телефону</span>
        </label>

        <label for="new_password">
          Створіть пароль(не менше 8 символів, має містити хоча б одну велику літеру і одну цифру):
          <input
              id="new_password"
              type="password"
              v-model="user.password"
              required
              @keyup="v$.password.$touch()"
          />
          <span v-if="v$.password.$error" class="error">Введіть коректний пароль</span>
        </label>
      </fieldset>
      <fieldset>
        <label for="male"><input id="male" type="radio" value="Чоловіча" v-model="user.sex" class="inline"
                                 required/>Чоловіча стать
        </label>

        <label for="female"><input id="female" type="radio" value="Жіноча" v-model="user.sex" class="inline"
                                   required/>Жіноча стать
        </label>

        <label for="othersex"><input id="othersex" type="radio" value="Інша" v-model="user.sex" class="inline"
                                     required/>Інше
        </label>
      </fieldset>
      <input class="button" type="submit" value="Створити акаунт"/>
    </form>
    <div class="container_button">
      <button id="delete-button" @click="deleteUsers">Видалити</button>
      <button id="duplicate-button" @click="duplicateUsers">Продублювати</button>
    </div>
    <table id="data_table">
      <thead>
      <tr>
        <th>✔️</th>
        <th>Ім'я</th>
        <th>Прізвище</th>
        <th>Email</th>
        <th>Номер телефону</th>
        <th>Стать</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(user, index) in users" :key="index">
        <td>
          <input type="checkbox" v-model="user.selected">
        </td>
        <td>{{ user.firstName }}</td>
        <td>{{ user.lastName }}</td>
        <td>{{ user.email }}</td>
        <td>{{ user.phone }}</td>
        <td>{{ user.sex }}</td>
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import {ref} from 'vue'
import {useVuelidate} from '@vuelidate/core'
import {required, maxLength, email, minLength} from '@vuelidate/validators'
import {IMaskDirective} from 'vue-imask';

const isUkrainePhoneNumber = (value) => {
  const phoneNumber = value.replace(/[^0-9]/g, '');
  const operatorPrefix = phoneNumber.substring(3, 5);

  const operators = {
    'Kyivstar': ['67', '68', '96', '97', '98'],
    'Vodafone': ['50', '66', '95', '99'],
    'Lifecell': ['63', '73', '93'],
  };

  if (!/^380\d{9}$/.test(phoneNumber)) {
    return false;
  }

  for (const operator in operators) {
    if (operators[operator].includes(operatorPrefix)) {
      return true;
    }
  }

  return false;
};
const isLetter = (value) => /^[А-ЯҐЄІЇ][а-яґєії'\s]+$/u.test(value);
const hasUpperCaseLetter = (value) => /[A-Z]/.test(value);
const hasDigit = (value) => /\d/.test(value);

export default {
  name: 'userUser',
  setup() {
    const user = ref({
      firstName: '',
      lastName: '',
      email: '',
      phone: '',
      password: '',
      sex: '',
      selected: false
    });

    const rules = {
      firstName: {required, isLetter, maxLength: maxLength(20)},
      lastName: {required, isLetter, maxLength: maxLength(30)},
      password: {required, hasUpperCaseLetter, hasDigit, minLength: minLength(8)},
      email: {required, email},
      phone: {required, isUkrainePhoneNumber},
    }

    const v$ = useVuelidate(rules, user)

    return {v$, user};
  },

  data() {
    return {
      users: JSON.parse(localStorage.getItem('users')) || [],
      phoneNumberMask: {
        mask: '+{38}(000)-000-00-00',
      }
    };
  },

  directives: {
    imask: IMaskDirective
  },

  methods: {
    onSubmit() {
      this.v$.$touch();
      if (!this.v$.$error) {
        this.registerUser();
        this.v$.$reset();
      }
    },
    registerUser() {
      let id = new Date().getTime();
      let newUser = {...this.user, id, selected: false};
      this.users.push(newUser);
      localStorage.setItem('users', JSON.stringify(this.users));
      this.user = {
        firstName: '',
        lastName: '',
        password: '',
        phone: '',
        sex: '',
        email: '',
        selected: false,
      };
    },
    deleteUsers() {
      this.users = this.users.filter(user => !user.selected);
      localStorage.setItem('users', JSON.stringify(this.users));
    },
    duplicateUsers() {
      this.users.forEach(user => {
        if (user.selected) {
          let id = new Date().getTime();
          let newUser = {...user, id, selected: false};
          this.users.push(newUser);
        }
      });
      localStorage.setItem('users', JSON.stringify(this.users));
    }
  }
}
</script>

<style>
body {
  width: 100%;
  height: 100vh;
  margin: 0;
  background-color: #1b1b32;
  color: #f5f6f7;
  font-family: Tahoma, serif;
  font-size: 16px;
}

h1 {
  margin: 1em auto 0;
  text-align: center;
}

form {
  width: 60vw;
  max-width: 500px;
  min-width: 300px;
  margin: 0 auto;
  padding-bottom: 2em;
}

fieldset {
  border: none;
  padding: 2rem 0;
  border-bottom: 3px solid #3b3b4f;
}


label {
  display: block;
  margin: 0.5rem 0;
}

input {
  margin: 10px 0 0 0;
  width: 100%;
  min-height: 2em;
}

input {
  background-color: #0a0a23;
  border: 1px solid #0a0a23;
  color: #ffffff;
}

.inline {
  width: unset;
  margin: 0 0.5em 0 0;
  vertical-align: middle;
}

input[type="submit"] {
  display: block;
  width: 60%;
  margin: 1em auto;
  height: 2em;
  font-size: 1.1rem;
  background-color: #3b3b4f;
  border-color: white;
  min-width: 300px;
}

#data_table {
  border-collapse: collapse;
  width: 100%;
}

#data_table th, #data_table td {
  border: 1px solid #411a62;
  text-align: center;
  padding: 5px;
}

#data_table th {
  background-color: #4861a2;
  font-weight: bold;
}

#delete-button, #duplicate-button {
  background-color: #1f3160;
  color: #ffffff;
  border: none;
  padding: 10px;
  margin: 10px;
  cursor: pointer;
}

#delete-button:hover, #duplicate-button:hover {
  background-color: #101d3d;
}

.error {
  color: red;
  font-size: 12px;
  margin-top: 5px;
}


@media screen and (max-width: 600px) {
  table, thead, tbody, th, td, tr {
    display: block;
    text-align: center;
  }

  #data_table th, #data_table td {
    border: 1px solid #ccc;
    text-align: center;
    padding: 5px;
  }

  thead {
    text-align: center;
  }

  thead tr {
    position: absolute;
    top: -9999px;
    left: -9999px;
  }

  tr {
    border: 1px solid #ccc;
  }

  .container_button {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  td:before {
    content: attr(data-label);
    border: none;
    border-bottom: 1px solid #eee;
    display: flex;
    justify-content: flex-start;
    padding-right: 10px;
  }

  td:nth-of-type(1):before {
    content: '✔️';
  }

  td:nth-of-type(2):before {
    content: "Ім'я";
  }

  td:nth-of-type(3):before {
    content: 'Прізвище';
  }

  td:nth-of-type(4):before {
    content: 'E-mail';
  }

  td:nth-of-type(5):before {
    content: 'Номер';
  }

  td:nth-of-type(6):before {
    content: 'Стать';
  }
}
</style>