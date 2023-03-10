<script setup>
import RatingStars from 'components/RatingStars.vue';
</script>
<template>
  <q-page-sticky position="top" :offset="[0, -50]">
    <div style="background: white; width: 100%">
      <h6 style="text-align: center">
        {{ 20 - ratingCount }} 개의 음식을 평가해보세요
      </h6>
      <q-linear-progress
        stripe
        rounded
        size="20px"
        :value="ratingCount / 20"
        color="warning"
        class="q-mt-sm"
      />
    </div>
  </q-page-sticky>

  <div style="overflow: auto; margin-top: 120px">
    <template v-for="(item, idx) in items" v-bind:key="idx">
      <Transition>
        <div class="row" v-if="item.show">
          <div class="box">
            <img class="box-image" :src="item.image" />
          </div>
          <div class="food-info" style="margin: 10px">
            <div>
              <p style="margin: 0 !important; font-size: 20px">
                {{ item.name }}
              </p>
            </div>
            <RatingStars
              v-on:rated="rated"
              :id="item.id"
              size="3em"
            ></RatingStars>
          </div>
        </div>
      </Transition>
    </template>
  </div>
</template>

<script>
import { defineComponent, ref } from 'vue';

export default defineComponent({
  name: 'InitialSignupFood',
  setup() {
    return {};
  },
  async mounted() {
    const foodInfo = await (
      await fetch('http://127.0.0.1:3000/DAO/SELECT', {
        method: 'POST',
        body: new URLSearchParams({
          columns: '*',
          table: 'food',
        }),
      })
    ).json();
    for (let i = 0; i < foodInfo.length; i++) {
      this.items[foodInfo[i].id] = {
        id: foodInfo[i].id,
        name: foodInfo[i].name,
        image: foodInfo[i].image,
        show: true,
      };
    }
  },
  data() {
    return {
      show: true,
      items: {
        /** sample {
         *  1 : {
         *    id: 1,
         *    name: '1',
         *    image: 'img',
         *    show: true
         *  }
         * }
         * */
      },
      ratingCount: 0,
      foodSet: new Set(),
    };
  },
  methods: {
    // MARK: 하위 템플릿(RatingStarts 에서 이벤트 emit)
    rated(rating, id) {
      // MARK: 20개의 음식을 모두 평가했다면 종료 아니라면 음식 추가
      this.ratingCount == 19 ? this.onSubmit() : this.ratingAdd(rating, id);
    },
    rand(min, max) {
      min = Math.ceil(min);
      max = Math.floor(max);
      return Math.floor(Math.random() * (max - min)) + min; //최댓값은 제외, 최솟값은 포함
    },
    onSubmit: async function () {
      await fetch('http://127.0.0.1:3000/DAO/UPDATE', {
        method: 'POST',
        body: new URLSearchParams({
          table: 'user',
          set: `initial_signin = 0`,
          where: `user_id = '${this.$q.cookies.get('user_id')}'`,
        }),
      });
      this.$router.push('/main/feed');
    },
    // MARK: 서버의 DAO 모듈 통해서 음식 평가 정보 INSERT
    ratingAdd: async function (rating, id) {
      // MARK: timeout 미실시시 렌더링된 템플릿이 없어 runtime error 발생
      setTimeout(async () => {
        await fetch('http://127.0.0.1:3000/DAO/INSERT', {
          method: 'POST',
          body: new URLSearchParams({
            table: 'rating',
            columns: `user_id, food_id, rating`,
            values: `${this.$q.cookies.get('user_key')}, ${id}, ${rating}`,
          }),
        });
        // MARK: Transition 템플릿 사용 평가 완료된 음식은 사라지도록 함
        this.items[id].show = false;
        this.ratingCount += 1;
      }, 200);
    },
  },
});
</script>

<style scoped>
.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}
.box {
  width: 80px;
  height: 80px;
  border-radius: 20%;
  overflow: hidden;
}
.box-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>
