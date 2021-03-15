<template>
    <app-layout>
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                <chat-room-selection
                    v-if="currentRoom.id"
                    :rooms="chatRooms"
                    :currentRoom="currentRoom"
                    v-on:roomchanged="setRoom($event)"
                />
            </h2>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                    <message-container :messages="messages" />
                    <input-message 
                        :room="currentRoom" 
                        v-on:messagesent="getMessage()"
                    />
                </div>
            </div>
        </div>
    </app-layout>
</template>

<script>
    import AppLayout from '@/Layouts/AppLayout'
    import MessageContainer from './messageContainer.vue'
    import InputMessage from './inputMessage.vue'
    import ChatRoomSelection from './chatRoomSelection.vue'

    export default {
        components: {
            AppLayout,
            MessageContainer,
            InputMessage,
            ChatRoomSelection
        },
        data: function() {
            return{
                chatRooms: [],
                currentRoom: [],
                messages: []
            }
        },
        watch: {
            currentRoom() {
                this.connect();
            }
        },
        methods: {
            connect() {
                if(this.currentRoom.id){
                    let vm = this;
                    this.getMessage();
                    window.Echo.private('chat.'+this.currentRoom.id)
                        .listen('.message.new', e => {
                            vm.getMessages();
                        })
                }
            },
            getRooms() {
                axios.get('/chat/rooms')
                    .then(res => {
                        this.chatRooms = res.data;
                        this.setRoom(res.data[0]);
                    })
                    .catch(err => {
                        console.log(err);
                    })
            },
            setRoom( room ) {
                this.currentRoom = room;
            },
            getMessage(){
                axios.get('/chat/room/'+this.currentRoom.id+'/messages')
                    .then( res => {
                        this.messages = res.data;
                    })
                    .catch(err => {
                        console.log(err);
                    })
            }
        },
        created(){
            this.getRooms()
        }
    }
</script>
