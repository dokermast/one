<template>
    <div class="row">
        <div class="col-sm-12">
            <textarea class="form-control" name="" id="" cols="30" rows="10" readonly="">
                {{ messages.join('\n')}}
            </textarea>
            <hr>
            <input class="form-control" type="text" v-model="textMessage" @keyup.enter="sendMessage">
        </div>
    </div>
</template>

<script>
    export default {
        name: "Chat",
        data() {
            return {
                messages: [],
                textMessage: ''
            }
        },
        mounted() {

            window.Echo.channel('chat').listen('Message', ({message}) => {
                this.messages.push(message);
            })
        },
        methods: {
            sendMessage() {
                axios.post('/messages', {body: this.textMessage}).then((response) => {

                    this.messages.push(this.textMessage);

                    this.textMessage = '';
                })
            }
        }
    }

</script>

<style scoped>

</style>
