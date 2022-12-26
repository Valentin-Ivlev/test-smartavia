<template>
  <div>
    <b-navbar type="dark" variant="dark">
      <b-navbar-brand href="#" class="ms-3"><span class="bg-white p-1" style="border-radius: 50%;"><b-icon icon="currency-bitcoin" variant="black"></b-icon></span> Генератор Bitcoin адресов и приватных ключей к ним</b-navbar-brand>
      <b-button ref="btn" size="sm" class="my-2 ml-auto me-3" @click="generate" variant="success">Сгенерировать</b-button>
    </b-navbar>
    <b-alert v-for="(item, i) in items" :key="i" :variant=item.variant class="m-3" dismissible fade show :ref="'loader_' + item.id">
      <h5 class="alert-heading">{{item.header}}</h5>
      <div class="text-center"><b-spinner variant="success" :class="item.spinner"></b-spinner>{{ item.message }}</div>
    </b-alert>
  </div>
</template>
<script>
import axios from "axios";
export default {
  data() {
    return {
      id_counter: 0,
      active_requests: 0,
      items: []
    }
  },
  watch: {
    active_requests: function () {
      const btn=this.$refs.btn;
      if (this.active_requests>=3) {
        btn.setAttribute('disabled', '');
      } else btn.removeAttribute('disabled');
    }
  },
  methods: {
    generate() {
      this.items.unshift({id: this.id_counter, variant: 'secondary', header: 'Генерация адреса и ключа', spinner: 'mx-auto d-block', message: ''});
      this.getaddrkey(this.id_counter);
      this.id_counter++;
      this.active_requests++;
    },
    getaddrkey(block_id) {
      const block=this.items.find(({ id })=>id===block_id);
      axios.get("btcaddr.php")
          .then(response => {
            console.log(response);
              if (response.data.addr) {
                block.variant = 'success';
                block.header='Адрес и ключ сгенерированы';
                block.message='Адрес: '+response.data.addr+', ключ: '+response.data.pkey;
              } else {
                block.variant = 'warning';
                block.header='Что-то пошло не так';
                if (response.data.error) block.message=response.data.error;
                else block.message=response.data;
              }
            })
          .catch(err => {
            block.variant='danger';
            block.header='Ошибка';
            if (err.response) block.message=err.response.status+' (Webmaster Is Drunk)';
            console.log(err);
          })
          .finally(() => {
            this.active_requests--;
            block.spinner='d-none';
          })
    }
  }
}
</script>