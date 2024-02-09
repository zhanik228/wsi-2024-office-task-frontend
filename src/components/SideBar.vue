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
                    <p>Being at work 3h 46m</p>
                </div>
                <button @click="changeAvatar" class="edit-button">Change avatar</button>
                <button @click="logout" class="end-button">End session</button>
            </div>
        </div>
        <div class="sidebar__body">
            <RoomList :sidebar_body="sidebar_body" />
        </div>
        <div class="sidebar__footer">
            <ChatRoom :token="token" :currentRoom="currentRoom" :messages="messages" />
        </div>
    </div>
    </div>
</template>

<script>
import axios from 'axios'
import pusher from 'pusher-js'
import Echo from "laravel-echo"

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
            sidebar_body: [
                {
                    title: {
                        name: 'The office',
                        bold: true,
                    },
                    people: [
                        'Mary',
                        'Richard'
                    ],
                    max_amount: 4,
                },
                {
                    title: {
                        name: 'Desk',
                        bold: false,
                    },
                    people: [
                        'John',
                    ],
                    max_amount: 3
                }
            ]
        };
    },
    methods: {
        changeAvatar() {
            this.$emit('update:modalMode', 'edit');
            this.$emit('update:modalOpen', true);
        },
        async getRooms() {
            try {
                const res = await axios.get('http://127.0.0.1:8000/api/v1/chat/rooms', {
                    headers: {
                        'Authorization': `Bearer ${this.token}`
                    }
                });
                this.chatRooms = res.data;
            }
            catch (error) {
                console.error(error);
            }
        },
        setRoom(room) {
            this.currentRoom = room;
        },
        async getUserRoom() {
            const res = await axios.get('http://127.0.0.1:8000/api/v1/user', {
                headers: {
                    'Authorization': `Bearer ${this.token}`
                }
            });
            console.log(res.data.chat_room_id);
            this.setRoom(this.chatRooms[res.data.chat_room_id - 1]);
            if (!res.data.chat_room_id) {
                try {
                    const res = await axios.put(`http://127.0.0.1:8000/api/v1/user/${this.user.id}`, {
                        chat_room_id: 1,
                    });
                    console.log(res);
                }
                catch (error) {
                    console.error(error);
                }
            }
        },
        async getMessages() {
            try {
                const res = await axios.get('http://127.0.0.1:8000/api/v1/chat/room/' + this.currentRoom.id + '/messages', {
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
                const res = axios.post('http://127.0.0.1:8000/api/v1/logout', {}, {
                    headers: {
                        'Authorization': `Bearer ${this.token}`
                    }
                });
                console.log(res);
                localStorage.removeItem('user');
                location.reload();
            }
            catch (error) {
                console.error(error);
            }
        },
        connect() {
            if (this.currentRoom.id) {
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
        this.getRooms();
        this.getUserRoom();
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
    background-color: rgb(152, 224, 224);
    height: 100vh;
    overflow-y: auto;
    z-index: 20;
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
}

.sidebar__footer {
    padding: 20px 10px;
}

</style>