<template>
  <div>
    <transition name="fade" appear>
      <PopupBackground
        v-if="showUpdate"
        @click="sendHideEvent"
      ></PopupBackground>
    </transition>

    <transition name="slide" appear>
      <PopupContainer v-if="showUpdate">
        <PopupTitle text="Alterar Dados da Pessoa"></PopupTitle>

        <Alert v-show="alert.show" :Alert="alert"></Alert>

        <UserForm
          :class="['ui', alert.type, 'form']"
          @submit.prevent="submitUpdate"
        >
          <FormLabel :class="{ error: inputErr.cpf }" text="CPF">
            <input
              placeholder="Insira seu CPF"
              type="text"
              v-model="user.cpf"
              v-mask="'###.###.###-##'"
              disabled
            />
          </FormLabel>

          <FormLabel :class="{ error: inputErr.name }" text="Nome">
            <input
              placeholder="Insira seu nome"
              type="text"
              maxlength="20"
              v-model="user.name"
            />
          </FormLabel>

          <FormLabel :class="{ error: inputErr.phone }" text="Telefone">
            <input
              placeholder="Insira seu Número"
              type="text"
              v-model="user.phone"
              v-mask="'(##) #####-####'"
            />
          </FormLabel>

          <ButtonsContainer>
            <CancelBtn @click="sendHideEvent"></CancelBtn>
            <SaveBtn
              type="submit"
              :disabled="btnDisabled"
              :class="{ loading: btnDisabled }"
            ></SaveBtn>
          </ButtonsContainer>
        </UserForm>
      </PopupContainer>
    </transition>
  </div>
</template>

<script>
// Components
import PopupBackground from "../popup/PopupBackground.vue";
import PopupContainer from "../popup/PopupContainer.vue";
import PopupTitle from "../popup/PopupTitle.vue";
import Alert from "../general/Alert.vue";
import UserForm from "../general/UserForm.vue";
import FormLabel from "../general/FormLabel.vue";
import ButtonsContainer from "../buttons/ButtonsContainer.vue";
import CancelBtn from "../buttons/CancelBtn.vue";
import SaveBtn from "../buttons/SaveBtn.vue";

// Composition API
import { onBeforeUpdate, ref } from "vue";

// Firestore
import db from "../../services/Firestore.js";
import { doc, getDoc, updateDoc } from "@firebase/firestore";

export default {
  components: {
    PopupBackground,
    PopupContainer,
    PopupTitle,
    Alert,
    UserForm,
    FormLabel,
    ButtonsContainer,
    CancelBtn,
    SaveBtn,
  },
  props: {
    userID: {
      typeof: Object,
      required: true,
    },
    showUpdate: {
      typeof: Boolean,
      required: true,
    },
  },
  setup(props) {
    async function getUser() {
      const userRef = doc(db, "users", props.userID);
      const userSnap = await getDoc(userRef);
      return userSnap.data();
    }

    const user = ref({
      name: "",
      cpf: "",
      phone: "",
    });

    onBeforeUpdate(async () => {
      await getUser().then((resp) => {
        user.value = { ...resp };
      });
    });

    return { user };
  },
  data() {
    return {
      inputErr: {
        name: false,
        cpf: false,
        phone: false,
      },
      alert: {
        show: false,
        type: "",
        title: "",
        message: "",
      },
      btnDisabled: false,
    };
  },
  methods: {
    sendHideEvent() {
      this.$emit("hideUpdate");
    },

    submitUpdate() {
      const user = this.user;

      if (this.userIsValid(user)) {
        this.resetAlert();

        this.btnDisabled = true;

        this.updateUserOnDatabase(user).then(() => {
          this.$emit("UserUpdated");
        });
      } else {
        this.verifyInputErr();
      }
    },

    userIsValid(user) {
      return (
        user.name.length >= 2 &&
        user.cpf.length == 14 &&
        user.phone.length == 15
      );
    },

    resetAlert() {
      this.inputErr = {
        name: false,
        cpf: false,
        phone: false,
      };
      this.alert = {
        show: false,
        type: "",
        title: "",
        message: "",
      };
    },

    async updateUserOnDatabase(user) {
      const userRef = doc(db, "users", user.cpf);

      await updateDoc(userRef, {
        name: user.name,
        phone: user.phone,
      });
    },

    showAlert(type, title, message) {
      this.alert.type = type;
      this.alert.title = title;
      this.alert.message = message;
      this.alert.show = true;
    },

    verifyInputErr() {
      const user = this.user;

      if (user.name.length < 2) {
        this.inputErr.name = true;
      } else {
        this.inputErr.name = false;
      }

      if (user.cpf.length != 14) {
        this.inputErr.cpf = true;
      } else {
        this.inputErr.cpf = false;
      }

      if (user.phone.length != 15) {
        this.inputErr.phone = true;
      } else {
        this.inputErr.phone = false;
      }

      if (this.inputErr.name || this.inputErr.cpf || this.inputErr.phone) {
        this.showAlert(
          "error",
          "Erro nos Dados!",
          "Preencha os seguintes campos corretamente."
        );
      }
    },
  },
};
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.5s;
}

.slide-enter-from,
.slide-leave-to {
  transform: translateY(100vh);
}
</style>