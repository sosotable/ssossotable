<template>
  <form class="form-signin" style="text-align: center">
    <div class="input-group mb-3">
      <div class="column justify-start">
        <q-input
          type="email"
          pattern="/^[a-zA-Z0-9.!#$%&’*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/"
          required
          outlined
          v-model="email"
          label="메일 주소를 입력해주세요"
          :hint="this.mail_duplication_message"
          :dense="dense"
          :rules="[
            (val) => this.mail_duplication_flag || '이미 가입된 이메일이에요',
          ]"
        />
        <q-btn
          color="white"
          @click="send_code"
          text-color="black"
          label="인증 번호 전송"
        />
      </div>
      <div class="column justify-start">
        <q-input
          outlined
          v-model="auth_code_confirm"
          label="인증 번호를 입력해주세요"
          :dense="dense"
          :hint="this.auth_code_confirm_message"
        />
      </div>
      <q-input
        outlined
        v-model="password_init"
        label="비밀번호를 입력해주세요"
        type="password"
        counter
        maxlength="20"
        minlength="6"
        :dense="dense"
        :rules="[
          (val) =>
            (val.length >= 6 && val.length <= 20) ||
            '6자에서 20자 사이의 비밀번호를 입력해주세요',
        ]"
      />
      <q-input
        outlined
        v-model="password_confirm"
        type="password"
        label="비밀번호를 다시 한 번 입력해주세요"
        :dense="dense"
        :hint="this.password_confirm_message"
      />
      <q-btn
        @click="signup"
        style="margin: 0 auto 20px auto"
        color="white"
        text-color="black"
        label="가입하기"
      />
    </div>
  </form>
  <q-dialog
    v-model="signup_submit_modal"
    persistent
    transition-show="flip-down"
    transition-hide="flip-up"
  >
    <q-card class="bg-primary text-white">
      <q-bar>
        <div><h6>환영해요!</h6></div>
        <q-space />
      </q-bar>
      <q-card-section />
      <q-card-section class="q-pt-none">
        소소식탁에 가입 완료되었어요!
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script>
import { defineComponent, ref } from 'vue';
import { io } from 'socket.io-client';
export default defineComponent({
  name: 'NormalSignUp',
  setup() {
    // MARK: 소켓 연결
    const socket = io('http://localhost:3000', {
      withCredentials: true
    });
    return {
      socket,
      dense: ref(false),
      signup_submit_modal: ref(false),
    };
  },
  data() {
    return {
      // MARK: 이메일 주소(user_id)
      email: '',
      // MARK: 인증 코드
      auth_code: '',
      auth_code_confirm: '',
      /**MARK: 메일 인증값
       * 인증 이후 바인딩된 email값 변경 방지 위한 값 할당
       * **/
      confirm_email: '',
      mail_duplication_flag: true,
      mail_duplication_message: '',
      auth_code_confirm_message: '',
      auth_code_status: false,
      password_init: '',
      password_confirm: '',
      /**MARK: 비밀번호 인증값
       * 인증 이후 바인딩된 password값 변경 방지 위한 값 할당
       * **/
      password_submit_confirm: '',
      password_confirm_message: '',
      password_flag: false,
    };
  },
  mounted() {
    this.socket.on('SELECT', (msg) => {
      if (msg.TAG === 'mail_duplication') {
        if (this.email.length > 0) {
          // MARK: 소켓 통신 통한 메일 중복성 검사
          msg.returnValue.length === 0
            ? (this.mail_duplication_message = '사용 가능한 이메일이에요')
            : (this.mail_duplication_flag = false);
        }
      }
    });
  },
  watch: {
    // MARK: email 값 변경 감지를 통한 해당 값 검증
    email: function () {
      if (this.mail_duplication_message !== '') {
        this.mail_duplication_message = '';
      }
      this.socket.emit('SELECT', {
        TAG: 'mail_duplication',
        columns: '*',
        table: 'user',
        where: `user_id = '${this.email}'`,
      });
    },
    // MARK: auth_code 값 검증
    auth_code_confirm: function () {
      parseInt(this.auth_code) === parseInt(this.auth_code_confirm)
        ? (this.auth_code_status = true)
        : (this.auth_code_status = false);
      if (this.auth_code_status) {
        this.auth_code_confirm_message = '인증 번호 확인!';
      } else {
        this.auth_code_confirm_message = '인증 번호가 일치하지 않아요';
      }
    },
    // MARK: password값 변경 감지를 통한 검증 (init)
    password_init: function () {
      parseInt(this.password_init) === parseInt(this.password_confirm) &&
      String(this.password_init).length <= 20 &&
      String(this.password_init).length >= 6
        ? (this.password_flag = true)
        : (this.password_flag = false);
      if (this.password_flag) {
        this.password_confirm_message = '비밀번호가 일치해요';
      } else {
        this.password_confirm_message = '비밀번호가 일치하지 않아요';
      }
    },
    // MARK: password값 변경 감지를 통한 검증 (confirm)
    password_confirm: function () {
      parseInt(this.password_init) === parseInt(this.password_confirm) &&
      String(this.password_init).length <= 20 &&
      String(this.password_init).length >= 6
        ? (this.password_flag = true)
        : (this.password_flag = false);
      if (this.password_flag) {
        this.password_confirm_message = '비밀번호가 일치해요';
      } else {
        this.password_confirm_message = '비밀번호가 일치하지 않아요';
      }
    },
  },
  methods: {
    // MARK: 인증 메일 전송
    async send_code() {
      // MARK: 이미 인증 코드를 보낸 경우
      if (this.send_code_status) {
        return;
      }
      // MARK: 이메일 유효성 확인
      if (
        this.email.match(
          /^[a-zA-Z0-9.!#$%&’*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/
        ) !== null
      ) {
        this.mail_duplication_message = '전송 완료!';
        this.send_code_status = true;
        this.confirm_email = this.email;
        this.auth_code = this.rand(999999, 100000);
        // MARK: 소소식탁 node.js 서버 경유한 naver mailer api 사용한 메일 전송
        await fetch('http://127.0.0.1:3000/authMail', {
          method: 'POST',
          body: new URLSearchParams({
            sendData: JSON.stringify({
              senderAddress: 'no_reply@ssossotable.com',
              title: '소소식탁 가입 인증 번호입니다',
              body: `${this.auth_code}`,
              recipients: {
                address: `${this.email}`,
                type: 'R',
              },
            }),
          }),
        });
        this.mail_duplication_flag = true;
      } else {
        this.mail_duplication_message = '올바른 메일 형식을 입력해주세요';
      }
    },
    // 회원가입
    signup: async function () {
      if (this.auth_code_status && this.password_flag) {
        this.signup_submit_modal = true;
        setTimeout(async () => {
          await fetch('http://127.0.0.1:3000/DAO/INSERT', {
            method: 'POST',
            body: new URLSearchParams({
              table: 'user',
              columns: 'user_id, user_password',
              values: `'${this.confirm_email}', '${this.password_init}'`,
            }),
          });
          this.$router.push('/');
        }, 2000);
      }
    },
    rand: function (max, min) {
      return Math.floor(Math.random() * (max - min) + min);
    },
  },
  computed: {},
});
</script>

<style scoped>
.q-field {
  margin: 20px;
}
button {
  margin: 0 20px;
}
</style>
