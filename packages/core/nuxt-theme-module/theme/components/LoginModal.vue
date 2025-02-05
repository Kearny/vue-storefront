<template>
  <SfModal
    v-e2e="'login-modal'"
    :visible="isLoginModalOpen"
    class="modal"
    @close="toggleLoginModal"
  >
    <template #modal-bar>
      <SfBar
        class="sf-modal__bar smartphone-only"
        :close="true"
        :title="isLogin ? 'Log in' : 'Sign in'"
        @click:close="toggleLoginModal"
      />
    </template>
    <transition name="sf-fade" mode="out-in">
      <div v-if="isLogin">
        <ValidationObserver v-slot="{ handleSubmit }" key="log-in">
          <form class="form" @submit.prevent="handleSubmit(handleLogin)">
            <ValidationProvider rules="required|email" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-email'"
                v-model="form.username"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="email"
                label="Your email"
                class="form__element"
              />
            </ValidationProvider>
            <ValidationProvider rules="required" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-password'"
                v-model="form.password"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="password"
                label="Password"
                type="password"
                class="form__element"
              />
            </ValidationProvider>
            <SfCheckbox
              v-e2e="'login-modal-remember-me'"
              v-model="rememberMe"
              name="remember-me"
              label="Remember me"
              class="form__element checkbox"
            />
            <div v-if="error.login">
              {{ error.login }}
            </div>
            <SfButton v-e2e="'login-modal-submit'"
              type="submit"
              class="sf-button--full-width form__button"
              :disabled="loading"
            >
              <SfLoader :class="{ loader: loading }" :loading="loading">
                <div>{{ $t('Login') }}</div>
              </SfLoader>
            </SfButton>
          </form>
        </ValidationObserver>
        <div class="action">
          <SfButton class="sf-button--text">
            {{ $t('Forgotten password?') }}
          </SfButton>
        </div>
        <div class="bottom">
          <p class="bottom__paragraph">{{ $t('No account') }}</p>
          <SfButton @click="setIsLoginValue(false)">
            {{ $t('Register today') }}
          </SfButton>
        </div>
      </div>
      <div v-else class="form">
        <ValidationObserver v-slot="{ handleSubmit }" key="sign-up">
          <form class="form" @submit.prevent="handleSubmit(handleRegister)" autocomplete="off">
            <ValidationProvider rules="required|email" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-email'"
                v-model="form.email"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="email"
                label="Your email"
                class="form__element"
              />
            </ValidationProvider>
            <ValidationProvider rules="required" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-firstName'"
                v-model="form.firstName"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="first-name"
                label="First Name"
                class="form__element"
              />
            </ValidationProvider>
            <ValidationProvider rules="required" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-lastName'"
                v-model="form.lastName"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="last-name"
                label="Last Name"
                class="form__element"
              />
            </ValidationProvider>
            <ValidationProvider rules="required" v-slot="{ errors }">
              <SfInput
                v-e2e="'login-modal-password'"
                v-model="form.password"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="password"
                label="Password"
                type="password"
                class="form__element"
              />
            </ValidationProvider>
            <ValidationProvider :rules="{ required: { allowFalse: false } }" v-slot="{ errors }">
              <SfCheckbox
                v-e2e="'login-modal-create-account'"
                v-model="createAccount"
                :valid="!errors[0]"
                :errorMessage="errors[0]"
                name="create-account"
                label="I want to create an account"
                class="form__element"
              />
            </ValidationProvider>
            <div v-if="error.register">
              {{ error.register }}
            </div>
            <SfButton
              v-e2e="'login-modal-submit'"
              type="submit"
              class="sf-button--full-width form__button"
              :disabled="loading"
            >
              <SfLoader :class="{ loader: loading }" :loading="loading">
                <div>{{ $t('Create an account') }}</div>
              </SfLoader>
            </SfButton>
          </form>
        </ValidationObserver>
        <div class="action">
          {{ $t('or') }}
          <SfButton v-e2e="'login-modal-login-to-your-account'" class="sf-button--text" @click="setIsLoginValue(true)">
            {{ $t('login in to your account') }}
          </SfButton>
        </div>
      </div>
    </transition>
  </SfModal>
</template>
<script>
import { ref, watch, reactive } from '@vue/composition-api';
import { SfModal, SfInput, SfButton, SfCheckbox, SfLoader, SfAlert, SfBar } from '@storefront-ui/vue';
import { ValidationProvider, ValidationObserver, extend } from 'vee-validate';
import { required, email } from 'vee-validate/dist/rules';
import { useUser } from '<%= options.generate.replace.composables %>';
import { useUiState } from '~/composables';

extend('email', {
  ...email,
  message: 'Invalid email'
});

extend('required', {
  ...required,
  message: 'This field is required'
});

export default {
  name: 'LoginModal',
  components: {
    SfModal,
    SfInput,
    SfButton,
    SfCheckbox,
    SfLoader,
    SfAlert,
    ValidationProvider,
    ValidationObserver,
    SfBar
  },
  setup() {
    const { isLoginModalOpen, toggleLoginModal } = useUiState();
    const form = ref({});
    const isLogin = ref(false);
    const createAccount = ref(false);
    const rememberMe = ref(false);
    const { register, login, loading, error: userError } = useUser();

    const error = reactive({
      login: null,
      register: null
    });

    const resetErrorValues = () => {
      error.login = null;
      error.register = null;
    };

    watch(isLoginModalOpen, () => {
      if (isLoginModalOpen) {
        form.value = {};
        resetErrorValues();
      }
    });

    const setIsLoginValue = (value) => {
      resetErrorValues();
      isLogin.value = value;
    };

    const handleForm = (fn) => async () => {
      resetErrorValues();
      await fn({ user: form.value });

      const hasUserErrors = userError.value.register || userError.value.login;
      if (hasUserErrors) {
        error.login = userError.value.login?.message;
        error.register = userError.value.register?.message;
        return;
      }
      toggleLoginModal();
    };

    const handleRegister = async () => handleForm(register)();

    const handleLogin = async () => handleForm(login)();

    return {
      form,
      error,
      userError,
      loading,
      isLogin,
      createAccount,
      rememberMe,
      isLoginModalOpen,
      toggleLoginModal,
      handleLogin,
      handleRegister,
      setIsLoginValue
    };
  }
};
</script>

<style lang="scss" scoped>

.modal {
  --modal-index: 3;
  --overlay-z-index: 3;
}
.form {
  margin-top: var(--spacer-sm);
  &__element {
    margin: 0 0 var(--spacer-xl) 0;
  }
}
.action {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: var(--spacer-xl) 0 var(--spacer-xl) 0;
  font: var(--font-weight--light) var(--font-size--base) / 1.6 var(--font-family--secondary);
  & > * {
    margin: 0 0 0 var(--spacer-xs);
  }
}
.action {
  margin: var(--spacer-xl) 0 var(--spacer-xl) 0;
}
.checkbox {
  margin-bottom: var(--spacer-2xl);
}
.bottom {
  text-align: center;
  margin-bottom: var(--spacer-lg);
  font-size: var(--h3-font-size);
  font-weight: var(--font-weight--semibold);
  font-family: var(--font-family--secondary);
  &__paragraph {
    color: var(--c-primary);
    margin: 0 0 var(--spacer-base) 0;
    @include for-desktop {
      margin: 0;
    }
  }
}
</style>
