<template>
  <div v-show="modalOpen" class="modal" @click="$emit('update:modalOpen', false)">
    <div class="modal-content" @click.stop>
      <div class="modal-header">
        <h2 class="modal-header__title">{{ modalMode == 'create' ? 'Create User' : 'Update User'}}</h2>
      </div>
      <div class="modal-body">
        <div class="modal-body__content">
          <div class="form">
            <div class="image-select">
              <img class="selected-avatar" :src="randomAvatar" alt="Selected Avatar">
            </div>
            <input :disabled="modalMode == 'edit'" v-model="username" required class="username-input" type="text" placeholder="Username">
            <span v-if="error" style="color: red">Username is required</span>
          </div>
          <div>
            <h2 class="avatars-title">You can choose any avatar you want</h2>
            <div class="avatars">
              <img class="choice-avatar" @click="selectedAvatar = avatar" v-for="avatar in avatars" :key="avatar" width="64" :src="avatar" alt="">
            </div>
          </div>
        </div>
          <button @click.prevent="onContinue" class="btn-continue">Continue</button>
      </div>
    </div>
  </div>
</template>

<script>
import firstAvatar from '../assets/avatars/avatar-01.svg'
import secondAvatar from '../assets/avatars/avatar-02.svg'
import thirdAvatar from '../assets/avatars/avatar-03.svg'
import fourthAvatar from '../assets/avatars/avatar-04.svg'
import fifthAvatar from '../assets/avatars/avatar-05.svg'
import sixthAvatar from '../assets/avatars/avatar-06.svg'
import seventhAvatar from '../assets/avatars/avatar-07.svg'
import eighthAvatar from '../assets/avatars/avatar-08.svg'
import ninthAvatar from '../assets/avatars/avatar-09.svg'
import tenthAvatar from '../assets/avatars/avatar-10.svg'
import eleventhAvatar from '../assets/avatars/avatar-11.svg'
import twelveAvatar from '../assets/avatars/avatar-12.svg'
import thirteenAvatar from '../assets/avatars/avatar-13.svg'
import fourteenAvatar from '../assets/avatars/avatar-14.svg'
import fifteenAvatar from '../assets/avatars/avatar-15.svg'
import sixteenAvatar from '../assets/avatars/avatar-16.svg'
import seventeenAvatar from '../assets/avatars/avatar-17.svg'
import eighteenAvatar from '../assets/avatars/avatar-18.svg'
import nineteenAvatar from '../assets/avatars/avatar-19.svg'

import axios from 'axios'

export default {
  data() {
    return {
      avatars: [
          firstAvatar,
          secondAvatar,
          thirdAvatar,
          fourthAvatar,
          fifthAvatar,
          sixthAvatar,
          seventhAvatar,
          eighthAvatar,
          ninthAvatar,
          tenthAvatar,
          eleventhAvatar,
          twelveAvatar,
          thirteenAvatar,
          fourteenAvatar,
          fifteenAvatar,
          sixteenAvatar,
          seventeenAvatar,
          eighteenAvatar,
          nineteenAvatar
      ],
      selectedAvatar: null,
      username: '',
      error: null,
    }
  },
  computed: {
    randomAvatar() {
      return this.selectedAvatar ? this.selectedAvatar : this.selectedAvatar = this.avatars[Math.floor(Math.random() * this.avatars.length)]
    },
  },
  watch: {
    modalMode(oldVal, newVal) {
      if (newVal == 'create') {
        const user = JSON.parse(localStorage.getItem('user'))
        this.selectedAvatar = `http://127.0.0.1:8000${user.avatar}`
        this.username = user.username
      }
    }
  },
  props: {
    modalMode: {
      required: false,
      type: String,
    },
    modalOpen: {
      required: false,
      type: Boolean
    }
  },
  methods: {
    async onContinue() {
      const selectedAvatar = this.selectedAvatar.split('/').pop()
      
      if (!this.username) {
        this.error = 'Username is required'
      }

      let res
      if (this.modalMode == 'create') {
        res = await axios.post('http://127.0.0.1:8000/api/v1/login', {
          username: this.username,
          avatar: selectedAvatar
        })
        console.log(res)
        localStorage.setItem('user', JSON.stringify(res.data.user))
        localStorage.setItem('token', JSON.stringify(res.data.token))
      } else {
        const user = JSON.parse(localStorage.getItem('user'))
        const res = await axios.put(`http://127.0.0.1:8000/api/v1/user/${user.id}`, {
          avatar: selectedAvatar
        })
        localStorage.setItem('user', JSON.stringify(res.data))
        console.log(res)
      }
      this.$emit('update:modalOpen', false)
      location.reload()
    }
  },
  mounted() {
  }
}
</script>

<style scoped>
.modal {
  z-index: 100;
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, .7);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: white;
  height: 700px;
  padding: 13px 15px;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
}

.modal-header {
  padding: 10px;
  text-align: center;
  border-bottom: 1px solid #ccc;
  margin-bottom: 10px;
}

.modal-header__title {
  font-weight: 700;
  font-size: 24px;
}

.modal-body {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}

.modal-body__content {
  display: flex;
  gap: 30px;
  flex-grow: 1;
}

.btn-continue {
  align-self: end;
  display: inline-block;
  padding: 10px 15px;
  background: rgba(84, 105, 212);
  border: none;
  border-radius: 4px;
  color: white;
  cursor: pointer;
}

.form {
  text-align: center;
  margin-bottom: 10px;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.username-input {
  border: none;
  border-bottom: 1px solid #ccc;
  background: #fff;
}

.username-input:focus {
  outline: none;
  border-bottom: 1px solid rgba(84, 105, 212);
}

.selected-avatar {
  background: rgba(0, 0, 0, .5);
  height: 120px;
  width: 120px;
}

.choice-avatar {
  cursor: pointer;
}

.choice-avatar:hover {
  transform: scale(1.1);
}

.avatars {
  display: grid;
  grid-template-columns: repeat(5, 64px);
  grid-template-rows: repeat(4, 64px);
  gap: 10px;
}

.avatars-title {
  font-size: 18px;
  margin: 10px 0;
}
</style>