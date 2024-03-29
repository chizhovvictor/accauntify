<template>
  <section>
    <div class="bg-color my-5">
      <div class="container py-1 bg-line">
        <div class="row my-5 justify-content-between py-5">
          <div class="col-12 col-lg-5">
            <div class="mb-4 text-white">
              <h2 class="mb-4 fs-1">
                {{ $t('form.title') }}
              </h2>
              <p class="mb-5 fs-5">
                {{ $t('form.description') }}
              </p>
            </div>
          </div>
          <div class="col-12 col-lg-6 col-xl-5">
            <div class="bg-white p-5 border-form shadow">
              <Input
                type="text"
                :label="$t('form.inputs.name.label')"
                id="name"
                :placeholder="$t('form.inputs.name.placeholder')"
                class="mb-4 required"
                v-model="name"
                :error="errors.name"
              />
              <Input
                type="tel"
                :label="$t('form.inputs.phone.label')"
                id="phone"
                :placeholder="$t('form.inputs.phone.placeholder')"
                class="mb-4 required"
                v-model="phone"
                v-maska:[options]
                :error="errors.phone"
              />
              <Button
                :text="$t('form.buttons.submit.name')"
                class="w-100 mb-3"
                @click="handleSubmit"
              />
              <p class="text-secondary text-center">
                <small>
                  {{ $t('form.buttons.submit.hint') }} <router-link
                    to="/privacy-policy"
                    class="text-secondary"
                  >
                    {{ $t('form.buttons.submit.link') }}
                  </router-link>
                </small>
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref } from "vue";
import Input from "@/components/Form/Input.vue";
import Button from "@/components/Form/Button.vue";
import { reactive } from "vue"
import {
    phoneMask,
    getMask,
    isEmpty,
    clearErrors,
    requiredError,
    minLengthError
} from "@/inputs";
import post from "@/request";
import { toast } from "vue3-toastify";
import { useI18n } from "vue-i18n";

const options = reactive({
    mask: (value) => phoneMask(value),
    eager: false,
});
const { t } = useI18n();

const name = ref(null);
const phone = ref(null);
const errors = ref({
    name: null,
    phone: null
});

const handleSubmit = async () => {
    clearErrors(errors);

    if (isEmpty(name.value)) {
        return requiredError(errors, 'name');
    }

    if (isEmpty(phone.value)) {
        return requiredError(errors, 'phone');
    }

    const countryCode = phone.value.split(' ')[0];
    const mask = getMask(countryCode);

    if (phone.value.length !== mask.length) {
        return minLengthError(errors, mask.length, 'phone')
    }

    try {
        await post('/api/telegram', {
            name: name.value,
            phone: phone.value
        });
        name.value = null;
        phone.value = null;
        toast.success(t('messages.success'));
    } catch (e) {
        toast.error(e.message);
    }
}

</script>

<style scoped>
.bg-color {
  background-image: linear-gradient(to right, #537379 100%, #B8D0D8 0%);
  background-size: cover;
}

.border-form {
  border-radius: 2rem;
}
.bg-line {
    background: url('../assets/line.svg') center center no-repeat;
    background-size: cover;
}
</style>