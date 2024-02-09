<template>
    <div class="chat">
        <h2 class="chat__title">{{ currentRoom.name }}</h2>
        <div class="chat-body">
            <p v-for="(message, index) in messages" :key="index">
                {{ message.user.username }}: {{ message.message }}
            </p>
        </div>
        <h2 class="chat-message__title">Send Message: </h2>
        <form style="display: flex;">
            <input v-model="message" @keyup.enter="sendMessage" type="text" placeholder="...">
            <button type="button" @click="sendMessage">Send</button>
        </form>
    </div>
</template>

<script>
import axios from 'axios';

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
        }
    },
    methods: {
        async sendMessage() {
            try {
                const res = await axios.post('http://127.0.0.1:8000/api/v1/chat/room/' + this.currentRoom.id + '/message', {
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
}

.chat {
    display: flex;
    flex-direction: column;
    gap: 10px;
}
</style>