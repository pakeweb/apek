<template>
  <div>
    <apek-title
      :subtitle="intro"
      :title="title"
    />
    <v-container grid-list-xl>
      <v-layout
        row
        wrap
        grid-list-md
      >

        <v-flex sm12 md6>
          <v-form
            @submit.prevent="send"
            v-model="form.valid"
            ref="contact"
            name="contact"
            column
            data-netlify="true"
            netlify-honeypot="bot-field"
            method="post"
          >
            <div style="display: none">
              <input name="bot-field">
            </div>
            <v-text-field
              label="Name"
              name="name"
              :rules="form.nameRules"
              v-model="form.name"
              autofocus
              autocomplete="off"
              clearable
              flex
              required
            />

            <v-text-field
              label="E-mail"
              autocomplete="off"
              name="email"
              v-model="form.email"
              :rules="form.emailRules"
              clearable
              required
            />

            <v-textarea
              label="Message"
              v-model="form.message"
              :rules="form.messageRules"
              name="message"
              rows="3"
              counter="160"
              clearable
              no-resize
              required
            />

            <v-layout row grid-list-md align-center justify-space-between class="mt-3">
              <vue-recaptcha
                sitekey="6LeN308UAAAAAPSx9gSXVD2HxgV4s3S0rqxhC8PG"
                ref="invisibleRecaptcha"
                @verify="onVerify"
                size="invisible"
                badge="inline"
              />
              <v-btn
                color="secondary"
                type="submit"
                :disabled="!form.valid"
              >
                Send
              </v-btn>
            </v-layout>
          </v-form>
        </v-flex>

        <v-flex>
          <v-list style="background: transparent">
            <v-list-tile>
              <v-list-tile-action>
                <!-- <v-icon>email</v-icon> -->
                <fa :icon="['fas', 'envelope']" size="lg"/>
              </v-list-tile-action>
              <v-list-tile-content v-text="email.join(', ')" />
            </v-list-tile>
            <v-list-tile>
              <v-list-tile-action>
                <fa :icon="['fas', 'phone']" size="lg" />
              </v-list-tile-action>
              <v-list-tile-content v-text="phone.join(', ')" />
            </v-list-tile>
            <v-list-tile>
              <v-list-tile-action>
                <fa :icon="['fas', 'fax']" size="lg" />
              </v-list-tile-action>
              <v-list-tile-content v-text="fax.join(', ')" />
            </v-list-tile>
            <v-list-tile>
              <v-list-tile-action>
                <fa :icon="['fas', 'building']" size="lg" />
              </v-list-tile-action>
              <v-list-tile-content>
                <v-list-tile-title v-text="address" />
                <v-list-tile-sub-title v-if="address2" v-text="address2" />
              </v-list-tile-content>
            </v-list-tile>
          </v-list>

          <no-ssr class="mt-4">
            <social-networks :networks="social" btn-class="mx-3" icon-size="2x"/>
          </no-ssr>
        </v-flex>
      </v-layout>
    </v-container>
    <v-snackbar
      bottom
      right
      v-model="snackbar"
    >
      Your message has been sent, thanks!
      <v-btn flat color="success" @click.native="clear">OK</v-btn>
    </v-snackbar>
  </div>
</template>

<script>
import ApekTitle from '~/components/ApekTitle';
import { VForm, VTextField, VSnackbar, VTextarea } from 'vuetify';
import SocialNetworks from '@/components/SocialNetworks';
import VueRecaptcha from 'vue-recaptcha';
import encode from '@/lib/encode';

export default {
  components: {
    SocialNetworks,
    VSnackbar,
    ApekTitle,
    VForm,
    VTextField,
    VTextarea,
    VueRecaptcha,
  },
  head() {
    return {
      title: this.title,
      script: [
        {
          src:
            'https://www.google.com/recaptcha/api.js?onload=vueRecaptchaApiLoaded&render=explicit',
          async: true,
          defer: true,
          body: true,
        },
      ],
    };
  },
  data: () => ({
    snackbar: false,
    form: {
      valid: false,
      name: '',
      nameRules: [v => !!v || 'Name is required'],
      email: '',
      emailRules: [
        v => !!v || 'E-mail is required',
        v =>
          /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v) ||
          'E-mail must be valid',
      ],
      message: '',
      messageRules: [v => !!v || 'Message is required'],
    },
  }),
  asyncData() {
    return import('~/content/pages/contact.json');
  },
  methods: {
    onVerify: function(uid) {
      const { email, name, message } = this.form;
      fetch('/', {
        method: 'POST',
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        body: encode({ 'form-name': 'contact', email, name, message, uid }),
      })
        .then(() => {
          this.snackbar = true;
        })
        .catch(error =>
          alert(
            'Unable to send message. Please use an alternative contact, thanks.'
          )
        );
    },
    resetRecaptcha() {
      this.$refs.recaptcha.reset(); // Direct call reset method
    },
    send() {
      if (this.$refs.contact.validate()) {
        this.$refs.invisibleRecaptcha.execute();
      }
    },
  },
};
</script>
