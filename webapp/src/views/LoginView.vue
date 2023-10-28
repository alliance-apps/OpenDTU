<template>
    <BasePage :title="$t('login.Login')" :isLoading="dataLoading">
        <BootstrapAlert v-model="showAlert" dismissible :variant="alertType">
            {{ alertMessage }}
        </BootstrapAlert>

        <CardElement :text="$t('login.SystemLogin')" textVariant="text-bg-danger">
            <form @submit.prevent="handleSubmit">
                <div class="form-group">
                    <label for="username">{{ $t('login.Username') }}</label>
                    <input type="text" v-model="username" name="username" class="form-control"
                        :class="{ 'is-invalid': submitted && !username }" @keydown.space.prevent />
                    <div v-show="submitted && !username" class="invalid-feedback">{{ $t('login.UsernameRequired') }}
                    </div>
                </div>
                <div class="form-group">
                    <label htmlFor="password">{{ $t('login.Password') }}</label>
                    <input type="password" v-model="password" name="password" class="form-control"
                        :class="{ 'is-invalid': submitted && !password }" />
                    <div v-show="submitted && !password" class="invalid-feedback">
                        {{ $t('login.PasswordRequired') }}</div>
                </div>
                <div class="form-group">
                    <button class="btn btn-primary" :disabled="dataLoading">{{ $t('login.LoginButton') }}</button>
                </div>
            </form>
        </CardElement>

        <CardElement :text="$t('configadmin.ResetHeader')" textVariant="text-bg-primary" center-content add-space>
            <button class="btn btn-danger" @click="onFactoryResetModal">{{ $t('configadmin.FactoryResetButton') }}
            </button>

            <div class="alert alert-danger mt-3" role="alert" v-html="$t('configadmin.ResetHint')"></div>
        </CardElement>

        <div class="modal" id="factoryReset" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">{{ $t('configadmin.FactoryReset') }}</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    {{ $t('configadmin.ResetMsg') }}
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" @click="onFactoryResetCancel"
                        data-bs-dismiss="modal">{{ $t('configadmin.Cancel') }}</button>
                    <button type="button" class="btn btn-danger" @click="onFactoryResetPerform">
                        {{ $t('configadmin.ResetConfirm') }}
                    </button>
                </div>
            </div>
        </div>
    </div>
    </BasePage>
</template>

<script lang="ts">
import * as bootstrap from 'bootstrap';
import { handleResponse } from '@/utils/authentication';
import BasePage from '@/components/BasePage.vue';
import BootstrapAlert from "@/components/BootstrapAlert.vue";
import CardElement from '@/components/CardElement.vue';
import router from '@/router';
import { login } from '@/utils';
import { defineComponent } from 'vue';

export default defineComponent({
    components: {
        BasePage,
        BootstrapAlert,
        CardElement,
    },
    data() {
        return {
            modalFactoryReset: {} as bootstrap.Modal,
            dataLoading: false,
            alertMessage: "",
            alertType: "info",
            showAlert: false,
            returnUrl: '',
            username: '',
            password: '',
            submitted: false,
        };
    },
    created() {
        // get return url from route parameters or default to '/'
        this.returnUrl = this.$route.query.returnUrl?.toString() || '/';
    },
    mounted() {
        this.modalFactoryReset = new bootstrap.Modal('#factoryReset');
    },
    methods: {
        onFactoryResetModal() {
            this.modalFactoryReset.show();
        },
        onFactoryResetCancel() {
            this.modalFactoryReset.hide();
        },
        onFactoryResetPerform() {
            const formData = new FormData();
            formData.append("data", JSON.stringify({ delete: true }));

            fetch("/api/config/delete", {
                method: "POST",
                //headers: authHeader(),
                body: formData,
            })
                .then((response) => handleResponse(response, this.$emitter, this.$router))
                .then(
                    (response) => {
                        this.alertMessage = this.$t('apiresponse.' + response.code, response.param);
                        this.alertType = response.type;
                        this.showAlert = true;
                    }
                )
            this.modalFactoryReset.hide();
        },
        handleSubmit() {
            this.submitted = true;
            const { username, password } = this;

            // stop here if form is invalid
            if (!(username && password)) {
                return;
            }

            this.dataLoading = true;
            login(username, password)
                .then(
                    () => {
                        this.$emitter.emit("logged-in");
                        router.push(this.returnUrl);
                    },
                    error => {
                        this.$emitter.emit("logged-out");
                        this.alertMessage = error;
                        this.alertType = 'danger';
                        this.showAlert = true;
                        this.dataLoading = false;
                    }
                )
        }
    }
});
</script>