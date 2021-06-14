<template>
  <div class="container">
    <div class="row">
      <div class="col-md-4">
        <div v-for="list in chatlist" :key="list.id">
          <div v-for="data in list.data.data" :key="data.id">
                 <button class="btn btn-sm btn-primary mb-2" @click="getMessage(data.room_id)">{{data}}</button>
          </div>
        </div>
      </div>
      <div class="col-md-8">
        <div class="d-flex flex-column align-items-stretch flex-shrink-0 bg-white">
      <div class="d-flex align-items-center flex-shrink-0 p-3 link-dark text-decoration-none border-bottom">
        <h4>Chat yuk borr</h4>
      </div>
      <div class="list-group list-group-flush border-bottom scrollarea">
        <div class="list-group-item list-group-item-action py-3 lh-tight"
             v-for="message in messages" :key="message.id"
        >
        <div class="card">
          <div class="card-body">
              <div class="d-flex w-100 align-items-center justify-content-between">
            <strong class="mb-1">{{ message.time }}</strong>
          </div>
          <div class="col-10 mb-1 small">{{ message.message }}</div>
          </div>
        </div>
        
        </div>
      </div>
    </div>
    <form @submit.prevent="submit">
      <input class="form-control" placeholder="Write a message" v-model="message"/>
    </form>
      </div>
    </div>
   
  </div>
</template>

<script>
import Pusher from 'pusher-js';

  const pusher = new Pusher('468f0c2d729723ab5378', {
      cluster: 'ap1',
       authEndpoint: `http://localhost:8000/api/chat/auth/`,
    })

export default {
  

  data() {
    return {
      id_room: '',
      username: 'username',
      message: '',
      messages: [],
      chatlist: [],
      messagesdata:[]
    }
  },

    //  async fetch() {
    //   this.chatlist = await fetch(
    //     'http://localhost:8000/api/listchat'
    //   ).then(res => res.json())
    // },

     async asyncData({$axios}) {
      const listmessage =  await $axios.get('http://localhost:8000/api/list/room/chat');
        
      return {listmessage}
    },


  mounted() {
    Pusher.logToConsole = true;

      // this.$echo.channel('chat.'+this.id_room)
      // .listen('message', (e) => {
      //     console.log(e);
      // });


   this.subscibeChannel()
    // const pusher = new Pusher('468f0c2d729723ab5378', {
    //   cluster: 'ap1',
    //   authEndpoint: 'http://localhost:8000/api/message'
    // });

  
    // const channel = pusher.subscribe(`private-auth-chat.1`);
      
    //  channel.bind(`chat-auth-events`, data => {
    //      this.message.push(data);
    //   })

    // var channel = pusher.subscribe('private-chat.'+this.id_room);
    // channel.bind('message', data => {
    //   this.messages.push(data);
    // });

    this.chatlist.push(this.listmessage)
    
  },

  methods: {
    
    subscibeChannel() {
      const channel = pusher.subscribe(`private-chat.${this.id_room}`);
      return channel
    },

    unsubscribeChannel() {
      pusher.unsubscribe(`private-chat.${this.id_room}`);
    },

    bindingChannel() {
      this.subscibeChannel().bind(`message`, data => {
        
        this.messages.push(data);

        this.chatlist.forEach(element => {
          element.data.data.forEach(i => {
            if(i.room_id==data.room_id){
              i.chat_message = data.message,
              i.time = data.time,
              i.is_read = data.is_read
            }
          })
          });
        
      })
    },

    async submit() {
      
    await this.$axios.post('http://127.0.0.1:8000/api/messages', {
           room_id : this.id_room,
          message: this.message,
      });
      this.message = '';
      // this.chatlist = []
      //const listmessage = await this.$axios.get('http://localhost:8000/api/list/room/chat');
      //this.chatlist.push(listmessage)
      this.bindingChannel()
     
    },

    async getMessage(id) {
         this.id_room = id.toString()
        this.unsubscribeChannel()
        this.subscibeChannel()
         
      const data = await this.$axios.get(`http://127.0.0.1:8000/api/chat/messages/${id}`);
      if(data.data.data.length===0){
         this.messages = []
      }else{
         this.messages = []
        data.data.data.forEach(element => {
            this.messages.push(element)
          });
      }
     
     
     // this.messages.push(data);
    //   this.message = '';
    },

    // getdata: function(){
    //     this.messages = []
    //   //  this.messages.push()

    //   this.datamessage.data.data.forEach(element => {
    //     this.messages.push(element)
    //   });

       

    //   //alert(this.messagesdata)
    // }
  }


}

</script>

<style>
.scrollarea {
  min-height: 500px;
}
</style>
