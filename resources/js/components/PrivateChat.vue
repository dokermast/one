<template>
    <div class="row">
        <div class="col-sm-8">
            <textarea class="form-control" name="" id="" cols="30" rows="10" readonly="">
                {{ messages.join('\n')}}
            </textarea>
            <hr>
            <input class="form-control" type="text" v-model="textMessage"
                   @keyup.enter="sendMessage" @keydown="actionUser">
            <span v-if="isActive">{{isActive.name}} typing a message ...</span>
        </div>
        <div class="col-sm-4">
            <ul>
                <li v-for="user in activeUsers">{{user}}</li>
            </ul>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Chat",
        props: ['room', 'user'],

        data() {
            return {
                messages: [],
                textMessage: '',
                isActive: false,
                typingTimer: false,
                activeUsers: []
            }
        },
        computed: {
            channel() {
                return window.Echo.join('room.' + this.room.id);
            }
        },
        mounted() {

            this.channel
                .here((users) => {
                    this.activeUsers = users;
                })
                .joining((users) => {
                    this.activeUsers.push(users);
                })
                .leaving((user) => {
                    this.activeUsers.splice(this.activeUsers.indexOf(user), 1);
                })
                .listen('PrivateChat', ({data}) => {
                    this.messages.push(data.body);
                    this.isActive = false;
                })
                .listenForWhisper('typing', (e) => {
                    this.isActive = e;

                    if(this.typingTimer) clearTimeout(this.typingTimer);

                    this.typingTimer = setTimeout(() => {
                        this.isActive = false;
                    }, 2000);
                })
        },
        methods: {
            sendMessage() {
                axios.post('/messages', {body: this.textMessage, room_id: this.room.id}).then((response) => {

                    this.messages.push(this.textMessage);

                    this.textMessage = '';
                })
            },
            actionUser() {
                this.channel
                    .whisper('typing', {
                        name: this.user.name
                    })
            }
        }
    }

</script>

<style scoped>

</style>

