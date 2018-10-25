<template>
  <div id="app">

    <!-- DIV Z GLOWNYM EKRANEM -->
    <div id="screen" ref="screen">

        <div v-for="response in responses" :key="response.id" class="row">

          <!-- DYMEK WIADOMOSCI -->
          <div v-if="!response.editMode" :class="'message-wrapper-' + response.type">
            
            <div v-show="response.type === 'response-right'" class="message-edit-container">
              <i class="fas fa-pencil-alt" @click="editResponse(response.id)"></i>
              <i class="fas fa-times" @click="deleteMessage(response.id)"></i>
            </div>

            <div :class="[response.type, {
            'last-message-right': response.isLastMessage && response.type === 'response-right',
            'last-message-left': response.isLastMessage && response.type === 'response-left',
            'first-message-right': response.isFirstMessage && response.type === 'response-right' && !response.isLastMessage,
            'first-message-left': response.isFirstMessage && response.type === 'response-left' && !response.isLastMessage,
            'last-first-left': response.isFirstMessage && response.type === 'response-left' && response.isLastMessage,          
            'last-first-right': response.isFirstMessage && response.type === 'response-right' && response.isLastMessage          
            }]">
              {{ response.message }}
            </div>
             
            <div v-show="response.type === 'response-left'" class="message-edit-container">
              <i class="fas fa-pencil-alt" @click="editResponse(response.id)"></i>
              <i class="fas fa-times" @click="deleteMessage(response.id)"></i>
            </div>
           
          </div>

          <!-- TRYB EDYCJI -->
          <div v-else>
            <div :class="[response.type + '-edit-message', response.type, {
            'last-message-right': response.isLastMessage && response.type === 'response-right',
            'last-message-left': response.isLastMessage && response.type === 'response-left',
            'first-message-right': response.isFirstMessage && response.type === 'response-right' && !response.isLastMessage,
            'first-message-left': response.isFirstMessage && response.type === 'response-left' && !response.isLastMessage,
            'last-first-left': response.isFirstMessage && response.type === 'response-left' && response.isLastMessage,          
            'last-first-right': response.isFirstMessage && response.type === 'response-right' && response.isLastMessage          
            }]">
              <textarea class="new-message-input" v-model="messageToEdit"></textarea>
              <div class="button-wrapper">
                <button class="add-button" @click="editMessage(response.id); response.editMode = false;">Zapisz</button>
                <button class="cancel-button" @click="response.editMode = false">Anuluj</button>
              </div>
            </div>
          </div>

          <!-- PODPIS -->
          <div v-show="!response.signatureEdit && !response.editMode">
            <div v-show="response.isLastMessage && !user" :class="response.type + '-signature'">
              <span v-if="response.type === 'response-left'" class="add-user" @click="editUser(response.id)">Uzytkownik</span>
              <span v-else>ecologic.io</span>
            </div>
            <div v-show="response.isLastMessage && user" :class="response.type + '-signature'">
              {{ response.type === 'response-right' ? 'ecologic.io' : user }}
            </div>
          </div>
          <div v-show="response.signatureEdit && !response.editMode">
            <input type="text" class="user-input response-left-signature" 
            :ref="'userInput' + response.id">
            <i class="fas fa-check icon" @click="saveUser(response.id)"></i>
            <i class="fas fa-times icon" @click="response.signatureEdit = false"></i>
          </div>
        </div>


        <!-- POLE TEKSTOWE NOWEJ WIADOMOSCI -->
        <div :class="['row', 'message-form', newMessageType + '-new-message']" v-show="showNewMessage">
          <textarea id="new-message-input" v-model="newMessage" @keyup.enter="newMessage += '<br>'" 
          placeholder="Wpisz nową wiadomość" ref="newMessage"></textarea>
          <div class="button-wrapper">
            <button class="add-button" @click="addMessage">Dodaj</button>
            <button class="cancel-button" @click="showNewMessage = false; newMessage = ''">Anuluj</button>
          </div>
        </div>

        <!-- PRZYCISKI DODAWANIA NOWYCH WIADOMOSCI -->
        <div class="row buttons" v-show="!showNewMessage">
          <div id="left-button" @click="toggleNewMessageEditor('response-left')">
          </div>
          <div id="right-button" @click="toggleNewMessageEditor('response-right')">
          </div>
        </div>

    </div>

    <!-- PRZYCISKI DO GENEROWANIA I RESETOWANIA KODU -->
    <div class="buttons-div">
      <button :class="['generate-button', {'generated': copied}]" @click="generateHtml"> {{ copied ? 'Skopiowano kod!' : 'Generuj kod' }} </button>
      <button class="reset-button" @click="reset">Resetuj</button>
    </div>

  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      responses: [],
      newMessage: '',
      showNewMessage: false,
      newMessageType: '',
      messageToEdit: '',
      user: '',
      createMode: true,
      generatedHtml: '',
      copied: false
    }
  },
  methods: {
    addMessage() { 
      this.responses.push({
        id: new Date().getTime().toString() + (Math.floor(Math.random() * 9999) + 1000),
        type: this.newMessageType,
        message: this.newMessage,
        editMode: false,
        isLastMessage: true,
        isFirstMessage: false,
        signatureEdit: false
      })
      this.showNewMessage = false;
      this.newMessage = '';
      this.addSignatures();
      this.updateFirstMessages();
      this.$refs.screen.scrollTop = this.$refs.screen.scrollHeight;
    },
    editMessage(id) {
      this.responses.find(response => response.id === id).message = this.messageToEdit;
      this.messageToEdit = '';
    },
    deleteMessage(id) {
      this.responses = this.responses.filter(response => response.id !== id);
      this.addSignatures();
      this.updateFirstMessages();
    },
    toggleNewMessageEditor(type){
      this.showNewMessage = true;
      this.newMessageType = type;
      this.$nextTick(() => {
        this.$refs.newMessage.focus();
      })
    },
    addSignatures() {
      this.responses.forEach((response, i) => {
        if(this.responses[i + 1]) {
          if (this.responses[i + 1].type !== response.type) {
            response.isLastMessage = true;
          } else {
            response.isLastMessage = false;
          }
        }
      })
      const lastMessage = this.responses[this.responses.length - 1];
      if (lastMessage) { lastMessage.isLastMessage = true };
    },
    updateFirstMessages() {
      this.responses.forEach((response, i) => {
        if (this.responses[i - 1]) {
          if (this.responses[i - 1].type !== response.type) {
            response.isFirstMessage = true;
          } else {
            response.isFirstMessage = false;
          }          
        }
      this.responses[0].isFirstMessage = true;
      })
    },
    editResponse(id) {
      this.responses.find(res => res.id === id).editMode = true; 
      this.messageToEdit = this.responses.find(res => res.id === id).message;
      this.responses.map(res => {
        if (res.id !== id) {
          let newRes = res;
          newRes.editMode = false;
          return newRes
        }
        return res
      })
    },
    generateHtml(){
      this.createMode = false;
      this.generatedHtml = '';
      const convertToHtml = () => {
        let output = '';
        this.responses.forEach(response => {
          if (response.type === 'response-left') {
            output += ` <div class="messages messages--received"> <div class="message">${response.message}</div></div>`
            if (response.isLastMessage) {
              output += `<div class="leftSign">${this.user}</div>`
            }
          }
          else {
            output += `<div class="messages messages--sent"> <div class="message">${response.message}</div></div>`
            if (response.isLastMessage) {
              output += `<div class="rightSign">ecologic.io</div>`
            }
          }
        })
        return output;
      };
      this.generatedHtml = `<!DOCTYPE html> <html> <head> <link href="https://fonts.googleapis.com/css?family=Roboto" rel="stylesheet"> <title>Konwersacja</title> <style>*{box-sizing: border-box;}.clearfix:after, .messages:after{content: ""; display: table; clear: both; height: 0; visibility: hidden;}body{background-color: #fff; height: 100%; font-family: 'Roboto', sans-serif;}.screen{background-color: #fff; height: 100%; width: 100%; margin: 0 auto; /* box-shadow: 0 2px 2px rgba(0, 0, 0, 0.2); */}.leftSign{float: left; color: black; margin-right: 70px; padding: 25px 25px 25px 25px; font-size: 33px; margin-bottom: 20px;}.rightSign{float: right; color: black; margin-left: 70px; padding: 25px 25px 25px 25px; font-size: 33px; margin-bottom: 20px;}.conversation{height: 100%; overflow: auto; padding: 25px; padding-bottom: 0;}.messages{/* margin-bottom: 20px; */}.messages--received .message{float: left; background-color: #ddd; border-bottom-left-radius: 5px; border-top-left-radius: 5px; margin-right: 70px; padding: 25px 25px 25px 25px;}.messages--received .message:first-child{border-top-left-radius: 15px;}.messages--received .message:last-child{border-bottom-left-radius: 15px;}.messages--sent .message{float: right; background-color: #1998e6; color: #fff; border-bottom-right-radius: 5px; border-top-right-radius: 5px; margin-left: 70px; padding: 25px 25px 25px 25px;}.messages--sent .message:first-child{border-top-right-radius: 15px;}.messages--sent .message:last-child{border-bottom-right-radius: 15px;}.message{display: inline-block; margin-bottom: 2px; clear: both; padding: 7px 13px; font-size: 38px; border-radius: 15px; line-height: 1.4;}.message--thumb{background-color: transparent !important; padding: 0; margin-top: 5px; margin-bottom: 10px; width: 20px; height: 20px; border-radius: 0px !important;}.text-bar{height: 50px; border-top: 1px solid #ccc;}.text-bar__field{float: left; width: calc(100% - 50px); height: 100%;}.text-bar__field input{width: 100%; height: 100%; padding: 0 20px; border: none; position: relative; vertical-align: middle; font-size: 24px;}.text-bar__thumb{float: left; width: 50px; height: 100%; padding: 10px;}.text-bar__thumb:hover{opacity: .8;}.text-bar__thumb .thumb{width: 100%; height: 100%; cursor: pointer;}.thumb{display: block;}.anim-wiggle{-webkit-animation: wiggle .2s ease infinite; animation: wiggle .2s ease infinite;}.anim-wiggle-2{-webkit-animation: wiggle2 .2s ease infinite; animation: wiggle2 .2s ease infinite;}@-webkit-keyframes wiggle{0%{-webkit-transform: rotateZ(5deg); transform: rotateZ(5deg);}50%{-webkit-transform: rotateZ(-5deg); transform: rotateZ(-5deg);}100%{-webkit-transform: rotateZ(5deg); transform: rotateZ(5deg);}}@keyframes wiggle{0%{-webkit-transform: rotateZ(5deg); transform: rotateZ(5deg);}50%{-webkit-transform: rotateZ(-5deg); transform: rotateZ(-5deg);}100%{-webkit-transform: rotateZ(5deg); transform: rotateZ(5deg);}}@-webkit-keyframes wiggle2{0%{-webkit-transform: rotateZ(10deg); transform: rotateZ(10deg);}50%{-webkit-transform: rotateZ(-10deg); transform: rotateZ(-10deg);}100%{-webkit-transform: rotateZ(10deg); transform: rotateZ(10deg);}}@keyframes wiggle2{0%{-webkit-transform: rotateZ(10deg); transform: rotateZ(10deg);}50%{-webkit-transform: rotateZ(-10deg); transform: rotateZ(-10deg);}100%{-webkit-transform: rotateZ(10deg); transform: rotateZ(10deg);}}.thumb-img, .thumb{background-size: contain; background-repeat: no-repeat;}</style> </head> <body> <div class="screen"> <div class="conversation"> ${convertToHtml()}</div></div></body> </html>`;
      setTimeout(() => { this.copied = true; }, 50);
      setTimeout(() => { this.copyHtml(); }, 200);
      setTimeout(() => { this.copied = false }, 1000);
    },
    reset() {
      this.responses = [];
      this.generatedHtml = '';
      this.newMessage = '';
      this.newMessageToEdit = '';
      this.user = '';
    },
    copyHtml() {
      const el = document.createElement('textarea');  
      el.value = this.generatedHtml;                                 
      el.setAttribute('readonly', '');               
      el.style.position = 'absolute';                 
      el.style.left = '-9999px';                      
      document.body.appendChild(el);                
      const selected =            
        document.getSelection().rangeCount > 0        
          ? document.getSelection().getRangeAt(0)     
          : false;                                    
      el.select();                                   
      document.execCommand('copy');                   
      document.body.removeChild(el);                  
      if (selected) {                                
        document.getSelection().removeAllRanges();    
        document.getSelection().addRange(selected);   
      }
    },
    saveUser(id) {
      this.user = this.$refs['userInput' + id][0].value;
      this.responses.find(res => res.id === id).signatureEdit = false;
    },
    editUser(id) {
      this.responses.find(res => res.id === id).signatureEdit = true;
      this.responses.map(res => {
        if (res.id !== id) {
          let newRes = res;
          newRes.signatureEdit = false;
          return newRes
        }
        return res
      })
      this.$nextTick(() => { this.$refs['userInput'+id][0].focus() });
    }
  }
}
</script>

<style lang="scss">
@import './scss/var';
*{
  ::-moz-selection { background: $blue; }
  ::selection { background: $blue; }
  box-sizing: border-box;
}
body, html, #app{
  font-family: $n;
  height: 100%;
  background-color: $lblue;
}

button, input, textarea {
  outline: none;
}

textarea {
  resize: none;
  border: none;
}

#app{
  padding-top: 35px;
}

#screen{
  margin: 0 auto;
  width: 400px;
  padding: 18px;
  height: calc(100% - 135px);
  border-radius: 20px;
  overflow-y: scroll;
  overflow-x: hidden;
  background-color: white;
  &::-webkit-scrollbar {
      width: 0px;
      background: transparent;
  }
  &:hover{
    .buttons{
      opacity: 1;
    }
  }
}

.row{
  width: 100%;
  clear: both;
  position: relative;
  &:hover{
    .edit-button, .delete-button {
      display: block;
    }
  }
}

.buttons{
  opacity: 0;
  margin-top: 10px;
  margin-bottom: 200px;
  transition: all 100ms ease-in-out;
}

#left-button{
  width: 30%;
  margin-right: 40%;
  height: 3em;
  float: left;
  cursor: pointer;
  border-radius: 17px 17px 17px 5px;
  transition: all 130ms cubic-bezier(.12,1.11,.46,.99);
  background-color: lighten($grey, 11);
  &:hover{
    background-color: $grey;
  }
}

#right-button{
  width: 30%;
  height: 3em;
  float: left;
  cursor: pointer;
  border-radius: 17px 17px 5px 17px;
  transition: all 130ms cubic-bezier(.12,1.11,.46,.99);
  background-color: lighten($blue, 45);
  &:hover{
    background-color: $blue;
  }
}

.buttons-div{
  margin: 35px auto 0 auto;
  width: 400px;
}

.generate-button{
  display: block;
  width: 280px;
  height: 40px;
  cursor: pointer;
  float: left;
  border-radius: 90px;
  border: none;
  color: $blue;
  background-color: white;
  letter-spacing: 0.5px;
  -webkit-box-shadow: 0px 3px 7px 1px rgba(0,124,201,0.1);
  -moz-box-shadow: 0px 3px 7px 1px rgba(0,124,201,0.1);
  box-shadow: 0px 3px 7px 1px rgba(0,124,201,0.1);
  transition: all 150ms cubic-bezier(.76,.18,.23,.96);
  &:hover{
    transform: translateY(-2px);
    -webkit-box-shadow: 0px 5px 8px 1px rgba(0,124,201,0.08);
    -moz-box-shadow: 0px 5px 8px 1px rgba(0,124,201,0.08);
    box-shadow: 0px 5px 8px 1px rgba(0,124,201,0.08);
  }
  &:active{
    transform: translateY(1px);
    -webkit-box-shadow: 0px 0px 0px 0px rgba(0,124,201,0.3);
    -moz-box-shadow: 0px 0px 0px 0px rgba(0,124,201,0.3);
    box-shadow: 0px 0px 0px 0px rgba(0,124,201,0.3);
  }
}

.reset-button{
  display: block;
  width: 100px;
  height: 40px;
  margin-left: 20px;
  cursor: pointer;
  float: left;
  border-radius: 90px;
  border: none;
  color: $red;
  background-color: white;
  letter-spacing: 0.5px;
  -webkit-box-shadow: 0px 3px 7px 1px rgba(0,124,201,0.1);
  -moz-box-shadow: 0px 3px 7px 1px rgba(0,124,201,0.1);
  box-shadow: 0px 3px 7px 1px rgba(0,124,201,0.1);
  transition: all 150ms cubic-bezier(.76,.18,.23,.96);
  &:hover{
    transform: translateY(-2px);
    -webkit-box-shadow: 0px 5px 8px 1px rgba(0,124,201,0.08);
    -moz-box-shadow: 0px 5px 8px 1px rgba(0,124,201,0.08);
    box-shadow: 0px 5px 8px 1px rgba(0,124,201,0.08);
  }
  &:active{
    transition: all 50ms cubic-bezier(.76,.18,.23,.96);
    transform: translateY(1px);
    -webkit-box-shadow: 0px 0px 0px 0px rgba(0,124,201,0.3);
    -moz-box-shadow: 0px 0px 0px 0px rgba(0,124,201,0.3);
    box-shadow: 0px 0px 0px 0px rgba(0,124,201,0.3);
  }
}

.message-wrapper-response-left{
  margin: 2px 0;
  text-align: left;
  &:hover{
    .message-edit-container{
      display: inline-block;
    }
  }
}

.message-wrapper-response-right{
  margin: 2px 0;
  text-align: right;
  &:hover{
    .message-edit-container{
      display: inline-block;
    }
  }
}

.response{
  height: auto;
  min-height: 100px;
  width: 100%;
}

.response-left{
  height: auto;
  max-width: 70%;
  display: inline-block;
  background-color: $grey;
  border-radius: 5px 17px 17px 5px;
  padding: 6px 12px;
}

.last-message-left{
  border-radius: 5px 17px 17px 17px !important;
}

.last-message-right{
  border-radius: 17px 5px 17px 17px !important; 
}

.first-message-left{
  border-radius: 17px 17px 17px 5px !important;
}

.first-message-right{
  border-radius: 17px 17px 5px 17px !important;
}

.last-first-left{
  border-radius: 17px !important;
}

.last-first-right{
  border-radius: 17px !important;
}

.response-right{
  height: auto;
  max-width: 70%;
  display: inline-block;
  background-color: $blue;
  border-radius: 17px 5px 5px 17px;
  color: white;
  padding: 6px 12px;
}

.message-form{
  padding: 10px;
  margin-bottom: 200px;
  margin-top: 10px;
  textarea{
    border-radius: 11px;
    margin-bottom: 10px;
    padding: 9px;
    color: white
  }
}

.response-left-new-message{
  background-color: $grey;
  border-radius: 17px 17px 17px 5px;
  float: left;
  ::-webkit-input-placeholder { color: rgba(0, 0, 0, 0.511) !important; }
  ::-moz-placeholder { color: rgba(0, 0, 0, 0.511) !important; }
  :-ms-input-placeholder { color: rgba(0, 0, 0, 0.511) !important; }
  :-moz-placeholder { color: rgba(0, 0, 0, 0.511) !important; }
  textarea{
    color: black;
    background-color: $grey;
  }
  button{
    background-color: lighten($grey, 5);
    color: black;
    &:hover{
      background-color: lighten($grey, 7);
    }
  }
}
.response-right-new-message{
  background-color: $blue;
  border-radius: 17px 17px 5px 17px;
  padding-bottom: 34px;
  ::-webkit-input-placeholder { color: rgba(255, 255, 255, 0.671) !important; }
  ::-moz-placeholder { color: rgba(255, 255, 255, 0.671) !important; }
  :-ms-input-placeholder { color: rgba(255, 255, 255, 0.671) !important; }
  :-moz-placeholder { color: rgba(255, 255, 255, 0.671) !important; }
  .button-wrapper{
    float: right;
  }
  textarea{
    background-color: $blue;
  }
  button{
    background-color: lighten($blue, 10);
    &:hover{
      background-color: lighten($blue, 12);
    }
  }
}

.response-left-edit-message{
  background-color: $grey;
  float: left;
  margin-bottom: 2px;
  ::-webkit-input-placeholder { color: rgba(0, 0, 0, 0.511) !important; }
  ::-moz-placeholder { color: rgba(0, 0, 0, 0.511) !important; }
  :-ms-input-placeholder { color: rgba(0, 0, 0, 0.511) !important; }
  :-moz-placeholder { color: rgba(0, 0, 0, 0.511) !important; }
  .button-wrapper{
    float: left;
  }
  textarea{
    color: black;
    background-color: $grey;
  }
  button{
    background-color: lighten($grey, 5);
    color: black;
    margin-left: 0;
    &:hover{
      background-color: lighten($grey, 7);
    }
  }
}
.response-right-edit-message{
  background-color: $blue;
  float: right;
  clear: both;
  margin-bottom: 2px;
  color: white !important;
  ::-webkit-input-placeholder { color: rgba(255, 255, 255, 0.671) !important; }
  ::-moz-placeholder { color: rgba(255, 255, 255, 0.671) !important; }
  :-ms-input-placeholder { color: rgba(255, 255, 255, 0.671) !important; }
  :-moz-placeholder { color: rgba(255, 255, 255, 0.671) !important; }
  .button-wrapper{
    float: right;
  }
  textarea{
    background-color: $blue;
    color: white !important;
  }
  button{
    margin-right: 0px;
    background-color: lighten($blue, 10);
    &:hover{
      background-color: lighten($blue, 12);
    }
  }
}

.message-edit-container{
  display: inline-block;
  line-height: 30px;
  color: $grey;
  display: none;
  i{
    cursor: pointer;
    margin: 0 1px;
    &:hover{
      color: $dgrey;
    }
  }
}

.edit-button{
  height: 30px;
  width: 30px;
  background-color: black;
  display: none;
}

.delete-button{
  height: 30px;
  width: 30px;
  background-color: red;
  display: none;
}

#user-input{
  position: fixed;
  top: 50px;
  left: 50px;
  width: 170px;
}

#new-message-input{
  width: 100%;
  height: 60px;
}

.response-left-signature{
  text-align: left;
  padding-top: 4px;
  color: $dgrey;
}

.response-right-signature{
  text-align: right;
  padding-top: 4px;
  color: $dgrey;
}

.add-button{
  margin: 0 5px;
  border-radius: 90px;
  border: none;
  font-size: 0.9em;
  line-height: 1.4em;
  padding: 2px 14px;
  color: white;
  cursor: pointer;
}

.cancel-button{
  margin: 0 5px;
  border-radius: 90px;
  border: none;
  font-size: 0.9em;
  line-height: 1.4em;
  padding: 2px 14px;
  color: white;
  cursor: pointer;
}

.user-input{
  border: none;
  color: $dgrey;
  width: 110px;
}

.icon{
  color: $grey;
  margin-left: 6px;
  cursor: pointer;
  &:hover{
    color: $dgrey;
  }
}

.add-user{
  cursor: pointer;
  &:hover{
    opacity: 0.7;
  }
}

.fa-pencil-alt{
  font-size: 0.8em !important;
  transform: translateY(-2px);
}

.generated{
  background-color: white;
  color: $green;
  transform: scale(1.08) translateY(-10px);
  -webkit-box-shadow: 0px 17px 18px 7px rgba(0,124,201,0.05);
  -moz-box-shadow: 0px 17px 18px 7px rgba(0,124,201,0.05);
  box-shadow: 0px 17px 18px 7px rgba(0,124,201,0.05);
  &:hover{
    transform: scale(1.08) translateY(-10px);
    -webkit-box-shadow: 0px 13px 18px 3px rgba(0,124,201,0.05);
    -moz-box-shadow: 0px 13px 18px 3px rgba(0,124,201,0.05);
    box-shadow: 0px 13px 18px 3px rgba(0,124,201,0.05);
  }
}

</style>
