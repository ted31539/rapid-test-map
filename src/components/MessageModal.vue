<template>
  <div
    ref="modal"
    class="modal fade border-0"
    id="exampleModal"
    tabindex="-1"
    aria-labelledby="exampleModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog  border-0">
      <div class="modal-content  border-0 shadow-lg">
        <div class="modal-header bg-warning border-0">
          <h5
            class="modal-title text-white text-shadow-white fw-bold"
            id="exampleModalLabel"
          >訊息</h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body d-flex flex-column align-items-center">
          <ul class="fs-5 text-shadow mb-6 fw-bold list-unstyled">
              <li v-for="text in messageText" :key="text" class="mb-4">{{ text }}
              </li>
          </ul>
          <button
            type="button"
            class="btn btn-warning text-white text-shadow-white fw-bold w-50"
            data-bs-dismiss="modal"
          >關閉</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Modal from 'bootstrap/js/dist/modal';

export default {
  data() {
    return {
      modal: '',
      messageText: [],
    };
  },
  props: {
    message: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  methods: {
    openModal() {
      this.modal.show();
    },
    closeModal() {
      this.modal.hide();
    },
  },
  watch: {
    message() {
      this.messageText = this.message;
    },
  },
  mounted() {
    this.modal = new Modal(this.$refs.modal, {
      backdrop: false,
    });
  },
};
</script>
