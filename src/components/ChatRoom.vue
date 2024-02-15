<template>
    <div class="chat">
        <h2 class="chat__title">{{ currentRoom?.name }}</h2>
        <div class="chat-body">
            <div :class="`msg ${message.user.id == currentUser.id ? 'my-msg' : ''}`" v-for="(message, index) in messages" :key="index">
                <h2 class="chat-header" :style="`color: ${message.user.color}`">
                    {{ message.user.username }}
                </h2>
                <div class="chat-message">{{ message.message }}</div>
            </div>
        </div>
        <h2 class="chat-message__title">Send Message: </h2>
        <form @submit.prevent="sendMessage">
            <input v-model="message" type="text" placeholder="...">
            <button type="submit">Send</button>
        </form>
    </div>
</template>

<script>
import axiosInstance from '@/api/axiosInstance'

export default {
    data() {
        return {
            message: ''
        }
    },
    props: {
        currentRoom: {
            required: false,
            type: Object
        },
        messages: {
            type: Array
        },
        token: {
            type: String
        },
        currentUser: {
            type: Object
        }
    },
    methods: {
        async sendMessage(e) {
            e.preventDefault()
            try {
                const res = await axiosInstance.post('http://127.0.0.1:8000/api/v1/chat/room/' + this.currentRoom.id + '/message', {
                    message: this.message
                }, {
                    headers: {
                        'Authorization': `Bearer ${this.token}`
                    }
                });
                if (res.status == 201) {
                    this.message = '';
                }
            }
            catch (error) {
                console.error(error);
            }
        },
    }
}
</script>

<style>
.chat-body {
    background: #fff;
    width: 100%;
    height: 250px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    padding: 5px;
}

.chat {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.msg {
    box-shadow: 1px 1px 1px 0 #ccc;
    border-radius: 15px;
    padding: 5px;
    align-self: flex-start;
}

.my-msg {
    align-self: flex-end;
}

.chat-header {
    font-size: 14px;
}
</style>