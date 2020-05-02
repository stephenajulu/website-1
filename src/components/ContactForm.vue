<template>
  <div
    class="form pb-40"
    refs>
    <div
      v-if="sent"
      class="success">
      <div class="flex items-center justify-start w-full">
        <div class="w-full p-10 mt-10 bg-gray-100 border-l-8 bg-accent border-accent">
          <h2>Message sent!</h2>
          <p class="m-0">
            Expect a response within the next 24 hours (though usually less).
          </p>
        </div>
      </div>
    </div>
    <form
      v-if="!sent"
      @submit.prevent="submit">
      <label
        for="message"
        class="block">
        <span
          class="text-gray-700">Message</span>
        <ClientOnly>
          <vue-expand
            id="message"
            v-model="form.message"
            :handler="handler"
            placeholder="Write your message here"
            min-row="5" />
        </ClientOnly>
      </label>
      <div>
        <div class="flex flex-col md:flex-row">
          <div class="md:w-1/2 md:mr-5">
            <label
              for="name"
              class="block mt-4">
              <span class="text-gray-700">Your name</span>
              <input
                id="name"
                v-model="form.name"
                name="name"
                class="block w-full mt-1 form-input"
                placeholder="Evan You"
                required>
            </label>
          </div>
          <div class="md:w-1/2">
            <label
              for="email"
              class="block mt-4">
              <span class="text-gray-700">Your email</span>
              <input
                id="email"
                v-model="form.email"
                name="email"
                class="block w-full mt-1 form-input"
                placeholder="evan@vuejs.org"
                required>
            </label>
          </div>
        </div>
        <vue-hcaptcha
          class="block mt-5 mb-2"
          :sitekey="hcaptchaKey"
          @verify="form.hcaptchaResponse = $event" />
        <button
          type="submit"
          :disabled="!form.hcaptchaResponse || submitting"
          :class="!form.hcaptchaResponse || submitting ? 'font-bold bg-gray-400 text-white' : 'font-bold bg-gray-800 text-accent'"
          class="block w-full px-10 py-6 mt-3 mb-5 text-xl font-bold rounded-md md:mt-8">
          <span v-if="!form.hcaptchaResponse">Please fill in the verification above</span>
          <span v-else-if="submitting">Sending your message...</span>
          <span v-else>Send it, Sparky!</span>
        </button>
        <p
          v-if="error"
          class="text-sm text-red-400 text-center mt-5">
          {{ context.error }}
        </p>
      </div>
    </form>
  </div>
</template>

<script>
// Components
import VueHcaptcha from '@hcaptcha/vue-hcaptcha'

// Packages
import ky from 'ky'
import Vue from 'vue'

// Mixins
import formMachineMixin from '@/mixins/formMachine'

// Environment variables
const hcaptchaKey = process.env.GRIDSOME_HCAPTCHA_SITE_KEY

export default {
  name: 'Form',
  components: {
    VueExpand: () => import('vue-expand'),
    VueHcaptcha
  },
  mixins: [formMachineMixin],
  data: () => ({
    hcaptchaKey,
    handler: new Vue(),
    form: { hcaptchaResponse: '' }
  }),
  mounted () {
    this.handler.$emit('focus')
  },
  formMachine: {
    resetTimeout: 5000
  },
  async submitHandler (payload) {
    const { status, message } = await ky.post('/api/contactSubmission', { json: payload }).json()
    if (status === 'error') throw new Error(message)
  },
  methods: {
    async submit () {
      this.formService.send({ type: 'SUBMIT', value: this.form })
    }
  }
}
</script>