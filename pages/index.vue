<template>
  <div class="container">
    <div class="row">
      <div class="col-md-4">
        <div v-for="list in chatlist" :key="list.id">
          <div v-for="data in list.data.data" :key="data.id">
                 <button class="btn btn-sm btn-primary mb-2" @click="getMessage(data.id)">{{data.user[0].name}}</button>
          </div>
        </div>
      </div>
      <div class="col-md-8">
        <div class="d-flex flex-column align-items-stretch flex-shrink-0 bg-white">
      <div class="d-flex align-items-center flex-shrink-0 p-3 link-dark text-decoration-none border-bottom">
        <input class="fs-5 fw-semibold" v-model="username"/>
      </div>
      <div class="list-group list-group-flush border-bottom scrollarea">
        <div class="list-group-item list-group-item-action py-3 lh-tight"
             v-for="message in messages" :key="message.id"
        >
          <div class="d-flex w-100 align-items-center justify-content-between">
            <strong class="mb-1">{{ message.user[0].name }}</strong>
          </div>
          <div class="col-10 mb-1 small">{{ message.text }}</div>
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
      const listmessage =  await $axios.get('http://localhost:8000/api/listchat');
      
      return {listmessage}
    },


  mounted() {
    Pusher.logToConsole = true;

    const pusher = new Pusher('468f0c2d729723ab5378', {
      cluster: 'ap1'
    });

    const channel = pusher.subscribe('chat');
    channel.bind('message', data => {
      this.messages.push(data);
    });

    this.chatlist.push(this.listmessage)

  },
  methods: {
    async submit() {
    await this.$axios.post('http://127.0.0.1:8000/api/messages', {
        data: {
           username: this.username,
           message: this.message,
           id_room : this.id_room,
        }
      });
      
      this.message = '';
    },

    async getMessage(id) {
        this.id_room = id
         this.messages = []
      const data = await this.$axios.get(`http://127.0.0.1:8000/api/getmessage/${id}`);
      if(data.data.data.length===0){
         this.messages = []
      }else{
        data.data.data.forEach(element => {
            this.messages.push(element)
          });

          console.log(this.messages)
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
