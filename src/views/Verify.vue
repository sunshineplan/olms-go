<template>
  <header style="padding-left: 20px">
    <a class="h3 title">{{ $t("VerifyRecord") }}</a>
    <hr />
  </header>
  <div class="form">
    <div class="form-row">
      <div class="form-group">
        <label for="department">{{ $t("Department") }}</label>
        <input
          class="form-control"
          :value="record.deptname"
          id="department"
          readonly
        />
      </div>
      <div class="form-group">
        <label for="employee">{{ $t("Employee") }}</label>
        <input
          class="form-control"
          :value="record.realname"
          id="employee"
          readonly
        />
      </div>
    </div>
    <div class="form-group">
      <label for="date">{{ $t("date") }}</label>
      <input
        class="form-control"
        :value="record.date.replace(':00Z', '').replace('T', ' ')"
        id="date"
        readonly
      />
    </div>
    <div class="form-group">
      <label for="type">{{ $t("type") }}</label>
      <input
        class="form-control"
        :value="record.type ? $t('overtime') : $t('leave')"
        id="type"
        readonly
      />
    </div>
    <div class="form-group">
      <label for="duration">{{ $t("duration") }}</label>
      <input
        class="form-control"
        :value="record.duration"
        id="duration"
        readonly
      />
    </div>
    <div class="form-group">
      <label for="describe">{{ $t("describe") }}</label>
      <textarea
        class="form-control"
        v-text="record.describe"
        id="describe"
        rows="3"
        readonly
      />
    </div>
    <button class="btn btn-success" @click="verify(true)">
      {{ $t("Accept") }}
    </button>
    <button class="btn btn-danger" @click="verify(false)">
      {{ $t("Reject") }}
    </button>
    <button class="btn btn-primary" @click="goback()">
      {{ $t("Cancel") }}
    </button>
  </div>
</template>

<script>
import Swal from "sweetalert2";

export default {
  name: "Verify",
  data() {
    return {
      record: this.$store.state.record,
    };
  },
  mounted() {
    document.title = this.$t("VerifyRecord") + " - " + this.$t("OLMS");
    window.addEventListener("keyup", this.cancel);
  },
  beforeUnmount() {
    window.removeEventListener("keyup", this.cancel);
  },
  methods: {
    async verify(status) {
      const data = { status };
      if (!status) {
        const comment = await Swal.fire({
          input: "textarea",
          inputLabel: this.$t("comment"),
          inputPlaceholder: this.$t("LeaveRejectedComment"),
          confirmButtonText: this.$t("OK"),
          cancelButtonText: this.$t("Cancel"),
          showCancelButton: true,
          customClass: {
            confirmButton: "swal btn btn-primary",
            cancelButton: "swal btn btn-primary",
          },
          buttonsStyling: false,
        });
        if (comment.isConfirmed) data.comment = comment.value;
        else return;
      }
      const resp = await this.post(
        `/record/verify/${this.record.id}`,
        data,
        "verify"
      );
      await this.checkResp(resp, async () => {
        await this.checkJson(await resp.json(), () => this.goback());
      });
    },
  },
};
</script>
