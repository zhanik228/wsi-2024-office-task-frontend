<template>
    <div>
        <div class="sidebar-toggle" @click="sidebarOpen = !sidebarOpen">
            <span class="bar"></span>
            <span class="bar"></span>
            <span class="bar"></span>
        </div>
    <div :class="`sidebar ${sidebarOpen ? 'sidebar_open' : ''}`">
        <div class="sidebar-arrow" @click="sidebarOpen = !sidebarOpen">&lt;</div>
        <div class="sidebar__header">
            <div class="avatar-container">
                <img width="120" height="120" :src="`http://127.0.0.1:8000${user.avatar}`" alt="avatar">
                <div class="avatar-description">
                    <h2 class="avatar-description__title">{{ user.username }}</h2>
                    <p>Being at work <span class="my-timer">00:00:00</span></p>
                </div>
                <button @click="changeAvatar" class="edit-button">Change avatar</button>
                <button @click="logout" class="end-button">End session</button>
            </div>
        </div>
        <div class="sidebar__body">
            <RoomList :sidebar_body="columns" />
        </div>
        <div class="sidebar__footer">
            <ChatRoom :token="token" :currentUser="user" :currentRoom="currentRoom" :messages="messages" />
        </div>
    </div>
    </div>
</template>

<script>
import axios from 'axios'
import pusher from 'pusher-js'
import Echo from "laravel-echo"
import axiosInstance from '@/api/axiosInstance'

import firstAvatar from '@/assets/avatars/avatar-01.svg'
import ChatRoom from './ChatRoom.vue'
import RoomList from './RoomList.vue'

export default {
    data() {
        return {
            message: '',
            chatRooms: [],
            currentRoom: [],
            messages: [],
            token: localStorage.getItem('user') ? JSON.parse(localStorage.getItem('token')) : '',
            user: localStorage.getItem('user') ? JSON.parse(localStorage.getItem('user')) : 'Username',
            sidebarOpen: false,
            avatar: firstAvatar,
        };
    },
    props: {
        columns: {
            required: false,
            type: Array
        },
        startTimer: {
            type: Boolean
        }
    },
    methods: {
        startTimerr() {
            const timer = document.querySelector('.my-timer').innerHTML
            const arr = timer.split(":")
            let hour = arr[0]
            let min = arr[1]
            let sec = arr[2]

            if (sec == 59) {
                if (min == 59) {
                    hour++
                    min = 0
                    if (hour < 10) hour = "0" + hour
                } else {
                    min++
                }
                if (min < 10) min = "0" + min;
                sec = 0
            } else {
                sec++
                if (sec < 10) sec = "0" + sec
            }
            document.querySelector('.my-timer').innerHTML = hour + ":" + min + ":" + sec
            setTimeout(this.startTimerr, 1000)
            localStorage.setItem('time',  hour + ":" + min + ":" + sec)
        },
        changeAvatar() {
            this.$emit('update:modalMode', 'edit');
            this.$emit('update:modalOpen', true);
        },
        setRoom(room) {
            this.currentRoom = room;
        },
        async getUserRoom() {
            const res = await axiosInstance.get('/api/v1/user', {
                headers: {
                    'Authorization': `Bearer ${this.token}`
                }
            });
            console.log(res)
            console.log(res.data.chat_room_id);
            if (res.data.chat_room_id) {
                this.setRoom(this.columns[res.data.chat_room_id - 1]);
            }
            else {
                try {
                    const res = await axios.put(`http://127.0.0.1:8000/api/v1/user/${this.user.id}`, {
                        chat_room_id: 1,
                    });
                    console.log(res);
                    this.setRoom(this.chatRooms[res.data.chat_room_id - 1])
                }
                catch (error) {
                    console.error(error);
                }
            }
        },
        async getMessages() {
            try {
                const res = await axiosInstance.get('/api/v1/chat/room/' + this.currentRoom.id + '/messages', {
                    headers: {
                        'Authorization': `Bearer ${this.token}`
                    }
                });
                this.messages = res.data;
            }
            catch (error) {
                console.error(error);
            }
        },
        async logout() {
            try {
                const res = axiosInstance.post('/api/v1/logout', {}, {
                    headers: {
                        'Authorization': `Bearer ${this.token}`
                    }
                });
                console.log(res);
                localStorage.removeItem('user');
                localStorage.removeItem('token')
                localStorage.removeItem('time')
                location.reload();
            }
            catch (error) {
                console.error(error);
            }
        },
        connect() {
            if (this.currentRoom?.id) {
                let vm = this;
                this.getMessages();
                window.Echo.private('chat.' + this.currentRoom.id)
                    .listen('.message.new', e => {
                    vm.getMessages();
                });
            }
        }
    },
    watch: {
        currentRoom() {
            this.connect();
        },
        columns() {
            this.getUserRoom();
        },
        startTimer(oldVal) {
            console.log(oldVal)
        }
    },
    mounted() {
        window.Pusher = pusher;
        window.Echo = new Echo({
            broadcaster: 'pusher',
            key: '22837aaac7ebdd96fa58',
            cluster: 'eu',
            forceTLS: true,
            authEndpoint: 'http://127.0.0.1:8000/broadcasting/auth',
            auth: {
                headers: {
                    Accept: 'application/json',
                    Authorization: `Bearer ${this.token}`
                }
            }
        });
        this.startTimerr()
    },
    components: { ChatRoom, RoomList }
}
</script>


<style scoped>
.sidebar-toggle {
    position: fixed;
    top: 10px;
    width: 50px;
    height: 30px;
    display: flex;
    gap: 4px;
    flex-direction: column;
    justify-content: space-between;
    margin: 20px;
    cursor: pointer;
    z-index: 2;
}

.bar {
    background: #000;
    height: 5px;
    width: 100%;
}

.sidebar {
    display: flex;
    flex-direction: column;
    position: fixed;
    left: 0;
    background-color: rgb(76, 195, 224);
    height: 100vh;
    overflow-y: auto;
    z-index: 100;
    transition: all .3s linear;
}

.sidebar-arrow {
    position: absolute;
    border: 1px solid red;
    right: 10px;
    cursor: pointer;
    display: none;
}


@media screen and (max-width: 720px) {
    .sidebar {
        left: -100%;
    }
    .sidebar-arrow {
        display: block;
    }
}
.sidebar_open {
    left: 0;
}

.sidebar__header {
    padding: 20px 10px;
    border-bottom: 1px solid #fff;
}

.avatar-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
}

.avatar-description__title {
    font-size: 24px;
}

.end-button {
    width: 100%;
    display: inline-block;
    padding: 5px 10px;
    background: rgb(212, 84, 105);
    border: none;
    border-radius: 4px;
    color: white;
    cursor: pointer;
}

.edit-button {
    width: 100%;
    display: inline-block;
    padding: 5px 10px;
    background: rgb(51, 91, 124);
    border: none;
    border-radius: 4px;
    color: white;
    cursor: pointer;
}

.sidebar__body {
    margin-top: 15px;
    border-bottom: 1px solid #fff;
    flex-grow: 1;
    padding: 10px 0;
}

.sidebar__footer {
    padding: 20px 10px;
}

</style>