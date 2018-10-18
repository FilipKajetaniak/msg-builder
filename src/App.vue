<template>
  <div id="app">

    <input id="user-input" type="text" v-model="user">

    <div id="screen" ref="screen">
        <div v-for="response in responses" :key="response.id" class="row">
          <div v-if="!response.editMode">
            <div :class="[response.type, {'last-message-right': response.isLastMessage && response.type === 'response-right',
            'last-message-left': response.isLastMessage && response.type === 'response-left'}]">
              {{ response.message }}
            </div>
            <div class="edit-button" @click="response.editMode = true; messageToEdit = response.message"></div>
            <div class="delete-button" @click="deleteMessage(response.id)"></div>
          </div>
          <div v-else>
            <div :class="[response.type, {'last-message-right': response.isLastMessage && response.type === 'response-right',
            'last-message-left': response.isLastMessage && response.type === 'response-left'}]">
              <textarea v-model="messageToEdit"></textarea>
              <button @click="editMessage(response.id); response.editMode = false;">Zapisz</button>
              <button @click="response.editMode = false">Anuluj</button>
            </div>
          </div>
          <div v-show="response.isLastMessage" :class="response.type + '-signature'">
            {{ response.type === 'response-right' ? 'ecologic.io' : user }}
          </div>
        </div>

        <div :class="['row', 'message-form', newMessageType + '-new-message']" v-show="showNewMessage">
          <textarea id="new-message-input" v-model="newMessage" @keyup.enter="newMessage += '<br>'"></textarea>
          <button @click="addMessage">Dodaj</button>
          <button @click="showNewMessage = false; newMessage = ''">Anuluj</button>
        </div>
        <div class="row buttons" v-show="!showNewMessage">
          <div id="left-button" @click="showNewMessage = true; newMessageType = 'response-left'">
          </div>
          <div id="right-button" @click="showNewMessage = true; newMessageType = 'response-right'">
          </div>
        </div>
    </div>
    <div class="buttons-div">
      <button class="generate-button" @click="generateHtml"> {{ copied ? 'Skopiowano kod!' : 'Generuj kod' }} </button>
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
        isLastMessage: true
      })
      this.showNewMessage = false;
      this.newMessage = '';
      this.addSignatures();
      this.$refs.screen.scrollTop = this.$refs.screen.scrollHeight;
    },
    editMessage(id) {
      this.responses.find(response => response.id === id).message = this.messageToEdit;
      this.messageToEdit = '';
    },
    deleteMessage(id) {
      this.responses = this.responses.filter(response => response.id !== id);
      this.addSignatures();
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
      setTimeout(() => { 
        this.copyHtml();
        this.copied = true;
      }, 100);
      setTimeout(() => { this.copied = false }, 2000);
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
    }
  }
}
</script>

<style lang="scss">
@import './scss/var';
// *{
// ::-moz-selection { background: $yellow; }
// ::selection { background: $yellow; }
// box-sizing: border-box;
// }
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
  width: 500px;
  padding: 18px;
  height: calc(100% - 135px);
  border-radius: 20px;
  // overflow-y: scroll;
  overflow: hidden;
  background-color: white;
  
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
  height: 100px;
  margin-bottom: 200px;
}

#left-button{
  width: 50%;
  height: 100px;
  float: left;
  cursor: pointer;
  border-radius: 17px 17px 17px 0;
  transition: all 130ms cubic-bezier(.12,1.11,.46,.99);
  &:hover{
    background-color: $grey;
  }
}

#right-button{
  width: 50%;
  height: 100px;
  float: left;
  cursor: pointer;
  border-radius: 17px 17px 0 17px;
  transition: all 130ms cubic-bezier(.12,1.11,.46,.99);
  &:hover{
    background-color: $blue;
  }
}

.buttons-div{
  margin: 35px auto 0 auto;
  width: 500px;
}

.generate-button{
  display: block;
  width: 380px;
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
  transition: all 130ms cubic-bezier(.12,1.11,.46,.99);
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
  transition: all 130ms cubic-bezier(.25,.99,.46,.84);
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

.response{
  height: auto;
  min-height: 100px;
  width: 100%;
}

.response-left{
  height: auto;
  min-height: 100px;
  width: 100%;
  background-color: $grey;
  border-radius: 17px;
  margin: 5px 0;
}

.last-message-left{
  border-radius: 17px 17px 17px 0 !important;
}

.last-message-right{
  border-radius: 17px 17px 0 17px !important; 
}

.response-right{
  height: auto;
  min-height: 100px;
  width: 100%;
  background-color: $blue;
  border-radius: 17px;
  margin: 5px 0;
}

.message-form{
  padding: 10px;
  textarea{
    border-radius: 11px;
    margin-bottom: 10px;
    padding: 9px;
    color: white
  }
}

.response-left-new-message{
  background-color: $grey;
  border-radius: 17px 17px 17px 0;
  float: left;
  textarea{
    background-color: lighten($grey, 7);
  }
}
.response-right-new-message{
  background-color: $blue;
  border-radius: 17px 17px 0 17px;
  textarea{
    background-color: lighten($blue, 10);
  }
  button{
      float: right;
  }
}

.edit-button{
  height: 30px;
  width: 30px;
  position: absolute;
  background-color: black;
  left: 10px;
  top: 10px;
  display: none;
}

.delete-button{
  height: 30px;
  width: 30px;
  position: absolute;
  background-color: red;
  left: 50px;
  top: 10px;
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
}

.response-right-signature{
  text-align: right;
}
</style>
