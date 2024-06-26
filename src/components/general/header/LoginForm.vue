<script setup>
import { ref } from 'vue';
import axios from 'axios';
import { useAuthStore } from "./../../../stores/AuthStore";
import { useRouter } from 'vue-router';
import ErrorPassword from '@/components/alerts/ErrorPassword.vue';

const router  = useRouter();

const props = defineProps({
  onClose: Function,
  registerClick: Function
});

const closeForm = () => {
  props.onClose();
};

const uri = import.meta.env.VITE_API_ENDPOINT_GENERAL;

const usernameInput = ref('');
const passwordInput = ref('');

const errorVisible = ref(false);

const authStore = useAuthStore();

const submitForm = async () => {
  try {
    const response = await axios.get(`${uri}/login`, {
      auth: {
        username: usernameInput.value,
        password: passwordInput.value
      },
      withCredentials: true
    });

    const data = response.data;
        
    console.log(response);

    authStore.userRole = data.roles;
    authStore.username = data.username;
    authStore.isAuthenticated = true;
        
    console.log(authStore.userRole, authStore.username, authStore.isAuthenticated);
    router.push('/');
    closeForm();
  } catch (error) {
    console.error(error);
    errorVisible.value = true;
  }
};

const registerForm = () => {
  props.registerClick();
};
</script>


<template>
  <div>

    <ErrorPassword :show="errorVisible" message="Error al iniciar sesión, contraseña o usuario incorrecto." />
    
    <div class="modal" @click="closeForm">
        <div class="modal_container" @click.stop>

          <section class="form_container">
                <form @submit.prevent="submitForm">
                  <div id="button_container">
                    <button class="mobile" @click.stop="closeForm">
                      <img src="/icons/icon-cross.svg" alt="cross icon">
                    </button>
                  </div>
                  <div class="form-content">
                    <h1>Iniciar Sesión</h1>
                    <article>
                      <div class="input_box">
                          <label>Nombre de usuario:</label>
                          <input v-model="usernameInput" type="text" placeholder="nombre de usuario" required>
                      </div>
                      <div class="input_box">
                          <label>Contraseña:</label>
                          <input v-model="passwordInput" type="password" placeholder="contraseña" required>
                      </div>
                      <div class="submit_container">
                          <button type="submit" >Enviar</button>
                      </div>
                    </article>
                  </div>
                </form>
          </section>
    
          <section id="welcome_image">
                <div id="button_container">
                  <button class="desk" @click.stop="closeForm">
                    <img src="/icons/icon-cross.svg" alt="cross icon">
                  </button>
                </div>
                <div id="images_container">
                  <img class="desk" src="/images/logo.svg" alt="">
                  <img src="/images/PrintGo.svg" alt="">
                  <p>Haciendo tangible lo inimaginable.</p>
                  <button @click="registerForm">¿Aún no tienes cuenta?</button>
                </div>
          </section>

        </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  height: 90vh;
  width: 100vw;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 900;

  .modal_container {
    height: 45rem;
    width: 80rem;
    display: flex;

    section {
      background-color: white;
      width: 50%;
      border-radius: 10px 0 0 10px;
    }

    #welcome_image {
      background-color: $primary-background;
      border-radius: 0 10px 10px 0;
      display: flex;
      justify-content: space-between;
      flex-direction: column;
      gap: 3rem;

      #images_container {
        display: flex;
        flex-direction: column;
        justify-content: space-around;
        align-items: center;
        height: 100%;
        font-size: 2rem;
        gap: 2rem;

        img {
          width: 30%;
        }

        p {
          color: $light-font;
          font-weight: 300;
        }

        button {
          padding: 1.3rem;
          border-radius: 10px;
          color: white;
          background-color: $primary-color;
        }
      }
    }

    .form_container {
      padding: 2rem;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 2rem;
    }

    form {
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column;
      gap: 3rem;
      height: 100%;
      width: 100%;

      .form-content {
        width: 100%;
        display: flex;
        flex-direction: column;
        gap: 2rem;

        h1 {
          font-size: 3rem;
          font-weight: 600;
          text-align: center;
        }

        > article {
          display: flex;
          flex-direction: column;
          align-items: center;
          gap: 2rem;
          
          button {
            color: white;
            padding: 1.3rem;
            background-color: $primary-color;
            text-align: center;
            width: 15rem;
            border-radius: 10px;
            font-size: 2rem;
          }
        } 
      }
    }

    .input_box {
      display: flex;
      flex-direction: column;
      gap: 0.5rem;

      label {
        font-size: 1.3rem;
      }

      input {
        padding: 0.5rem 1rem;
        font-size: 1.8rem;
        border: 1px solid gray;
        border-radius: 5px;
        width: 100%;
      }
    }

    #button_container {
      width: 100%;
      display: flex;
      justify-content: end;
      
      button {
        margin: 2rem 2rem 0 0;
      }
      img {
        width: 2.5rem;
      }
    }
  }
}

.mobile {
  display: none;
}

.desk {
    display: block;
  }

@media (max-width: 1000px) {

  .mobile {
    display: block;
  }

  .desk {
    display: none;
  }

  .modal {
    max-height: 100vh;
  }

  .modal .modal_container {
    height: 90vh;
    width: 100vw;
    flex-direction: column;
    background-color: white;

    section {
      width: 100%;
      height: 60%;
      border-radius: 0;
    }

    #welcome_image {
      gap: 0;
      justify-content: center;
      border-radius: 10px 10px 0 0;
      padding-bottom: 0;
      height: 40%;
      
      #images_container {

        p {
          font-size: 1.3rem;
        }

        button {
          font-size: 1.3rem;
          padding: 0.5rem;
          height: 3rem;
        }

        img {
          width: 10rem;
        }
      }
    }
  }

  .modal .modal_container form .form-content > article button {
    font-size: 1.3rem;
    height: 100%;
    padding: 0.5rem
  }

  .modal .modal_container .input_box input {
    font-size: 1.3rem;
  }

  .modal .form_container form {
    align-items: center;
    justify-content: center;
    flex-direction: column;
  }
}
</style>