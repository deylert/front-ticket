<template>
    <v-card elevation="6"  class="mx-5" style=" max-height: 90vh;">
      <!-- Vista de Chat -->
      <v-row justify="center" class="mx-5">
        <v-col cols="12" md="12">
          <v-card-title class="d-flex justify-space-between align-center" color="#03626C" dark>
            <span class="text-h6">Chat en vivo</span>
          </v-card-title>
          <v-card-text>
            <v-container ref="chatContainer" style="overflow-y: auto; max-height: 450px;">
              <v-list>
                <!-- Iterar sobre los mensajes -->
                <v-list-item
                  v-for="(message, index) in chatMessages"
                  :key="index"
                  class="message-item"
                  :class="{ 'user-message-container': message.from === 'user' }"
                >
                  <v-row align="center">
                    <!-- Avatar de IA -->
                    <v-col v-if="message.from === 'ai'" class="d-flex justify-start" cols="auto">
                      <v-avatar color="#03626C">
                        <img src="@/assets/logo-blanco.png" alt="Imagen de avatar" class="avatar-image"/>
                      </v-avatar>
                    </v-col>
  
                    <!-- Mensaje -->
                    <v-col class="message-text" :class="{ 'text-right': message.from === 'user' }">
                      <v-card
                        :class="message.from === 'ai' ? 'ai-message' : 'user-message'"
                        elevation="2"
                        style="display: inline-block; max-width: 100%;"
                      >
                        <v-card-text style="white-space: pre-wrap;">{{ message.text }}</v-card-text>
                      </v-card>
                      <v-list-item-subtitle class="text-caption">
                        {{ message.timestamp }}
                      </v-list-item-subtitle>
                    </v-col>
  
                    <!-- Avatar de Usuario -->
                    <v-col v-if="message.from === 'user'" class="d-flex justify-end" cols="auto">
                      <v-avatar color="#DA7171">
                        <v-icon>mdi-account</v-icon>
                      </v-avatar>
                    </v-col>
                  </v-row>
                </v-list-item>
              </v-list>
            </v-container>
          </v-card-text>
  
          <!-- Campo de texto para escribir mensajes -->
          <v-divider></v-divider>
          <v-card-actions>
            <v-text-field
              v-model="newMessage"
              label="Escribe tu mensaje"
              append-icon="mdi-send"
              @keyup.enter="sendMessage"
              dense
              variant="underlined"
              full-width
            ></v-text-field>
          </v-card-actions>
        </v-col>
      </v-row>
  
      <!-- Snackbar para mensajes de alerta -->
      <v-snackbar
        v-model="snackbar"
        :timeout="sb_timeout"
        :color="sb_type"
        elevation="24"
        bottom
      >
        <v-row>
          <v-col md="2">
            <v-avatar :icon="sb_icon" color="sb_type" size="40"></v-avatar>
          </v-col>
          <v-col md="10">
            <h4>{{ sb_title }}</h4>
            {{ sb_message }}
          </v-col>
        </v-row>
      </v-snackbar>
    </v-card>
  </template>
  
  
  <script>
  export default {
    data() {
      return {
        chatMessages: [
          { from: 'ai', text: 'Hola, ¿cómo te encuentras hoy, En que puedo ayudarte?', timestamp: '12:00 PM' },
          { from: 'user', text: 'Me siento un poco cansado y me preocupa mi nivel de azúcar. ¿Qué debería hacer?', timestamp: '12:01 PM' },
          { from: 'ai', text: 'Es importante controlar los niveles de azúcar. Te recomendaría hacer ejercicio moderado...', timestamp: '12:03 PM' },
          { from: 'user', text: '¿Hay algo más que pueda hacer de forma natural?', timestamp: '12:07 PM' },
          { 
        from: 'ai', 
        text: 'Claro, el cansancio puede deberse a varios factores como estrés, falta de sueño o una mala alimentación. Es importante identificar la causa, pero mientras tanto, puedes probar remedios naturales como un té de manzanilla con miel antes de dormir, ya que ayuda a relajarte y mejorar el descanso. Además, incluir alimentos ricos en magnesio como plátanos o nueces durante el día puede ser beneficioso para reducir la fatiga.', 
        timestamp: '12:08 PM' 
        }
        ],
        newMessage: '',
        snackbar: false,
        sb_type: 'success',
        sb_message: '',
        sb_timeout: 3000,
        sb_title: '',
        sb_icon: 'mdi-check-circle',
      };
    },
    methods: {
      sendMessage() {
        if (this.newMessage.trim()) {
          // Agregar el mensaje del usuario
          this.chatMessages.push({ from: 'user', text: this.newMessage, timestamp: new Date().toLocaleTimeString() });
          // Simular respuesta de IA
          setTimeout(() => {
            this.chatMessages.push({ from: 'ai', text: 'Gracias por tu mensaje. ¿Necesitas ayuda con algo más?', timestamp: new Date().toLocaleTimeString() });
            this.showAlert('success', 'Mensaje enviado correctamente', 2000);
            this.scrollToBottom();
          }, 1000);
          this.newMessage = ''; // Limpiar campo de mensaje
        }
      },
      showAlert(type, message, timeout) {
        this.sb_type = type;
        this.sb_message = message;
        this.sb_timeout = timeout;
        this.sb_icon = type === 'success' ? 'mdi-check-circle' : 'mdi-alert-circle';
        this.snackbar = true;
      },
      scrollToBottom() {
        this.$nextTick(() => {
          const container = this.$refs.chatContainer;
          container.scrollTop = container.scrollHeight;
        });
      },
    },
  };
  </script>
  
  <style scoped>
  .avatar-image {
  width: 100%;
  height: 100%;
  object-fit: cover; /* Ajusta la imagen para que cubra el contorno del avatar */
  border-radius: 50%; /* Asegura que la imagen sea circular */
}
  .message-item {
  margin-bottom: 5px;
}

.user-message-container {
  flex-direction: row-reverse; /* Invertir el orden de avatar y mensaje */
}

.message-text {
  max-width: 100%; /* Limitar el ancho máximo de los mensajes */
}

.user-message {
  background-color: #DA7171;
  border-radius: 12px;
  padding: 1px;
  font-size: 14px;
  color: white;
}

.ai-message {
  background-color: #03626C;
  border-radius: 12px;
  padding: 1px;
  font-size: 14px;
  color: white;
}

.v-card-text {
  line-height: 1.5; /* Mejorar la separación entre líneas */
}

  </style>
  