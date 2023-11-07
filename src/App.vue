<script setup>
import { ref, reactive, computed } from 'vue'
import axios from 'axios'
import moment from 'moment'

const review = reactive({
  author: '',
  rating: null,
  text: '',
  // photo: null,
  created_at: null,
})

const convertToBase64 = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.readAsDataURL(file)
    reader.onload = () => resolve(reader.result)
    reader.onerror = (error) => reject(error)
  })
}

const reviewList = ref([])

// const uploadPhoto = (e) => {
//   const [file] = e.target.files
//   review.photo = file
// }

const setTimeAgo = (timestamp) => {
  return moment.unix(timestamp).fromNow()
}

const parseLinks = (text) => {
  const urlRegex = /(https?:\/\/[^\s]+)/g
  return text.replace(urlRegex, (url) => {
    return `<a href="${url}" target="_blank">${url}</a>`
  })
}

const submit = async () => {
  // if (review.photo) {
  //   review.photo = await convertToBase64(review.photo)
  // }

  review.created_at = moment().unix()

  axios
    .post('http://127.0.0.1:3000/api/reviews', review, {
      headers: {
        'Content-Type': 'application/json',
      },
    })
    .then((res) => {})
    .catch((err) => {
      console.log(err)
    })
    .finally(() => {
      window.location.reload()
    })
}

const reviews = () => {
  axios
    .get('http://127.0.0.1:3000/api/reviews')
    .then((res) => {
      reviewList.value = res.data.reviews
    })
    .catch((err) => {
      console.log(err)
    })
}
reviews()
</script>

<template>
  <Transition appear>
    <form @submit.prevent.stop="submit" class="container mt-4 pt-3 pb-3">
      <h2 class="mb-4">Vue Review</h2>

      <input
        v-model="review.author"
        placeholder="Author..."
        class="form-control"
        type="text"
        required
      />

      <textarea
        v-model="review.text"
        class="form-control mt-3"
        placeholder="Review..."
        rows="4"
        required
      ></textarea>

      <div class="mt-3">
        <label class="form-label">Rating</label>
        <select required v-model="review.rating" class="form-select">
          <option value="⭐">⭐</option>
          <option value="⭐⭐">⭐⭐</option>
          <option value="⭐⭐⭐">⭐⭐⭐</option>
          <option value="⭐⭐⭐⭐">⭐⭐⭐⭐</option>
          <option value="⭐⭐⭐⭐⭐">⭐⭐⭐⭐⭐</option>
        </select>
      </div>

      <!-- <div class="mt-3">
        <label class="form-label">Photo</label>
        <input
          @change="uploadPhoto"
          class="form-control"
          type="file"
          accept="image/*"
        />
      </div> -->

      <button class="btn btn-primary w-100 mt-3">Submit</button>
    </form>
  </Transition>

  <div class="container mb-5 mt-5">
    <h3 class="d-flex align-items-center mb-4">
      All Reviews
      <span
        class="badge fs-4 ms-2 bg-dark-subtle border user-select-none border-dark-subtle bg-opacity-25 review__length"
      >
        {{ reviewList.length }}
      </span>
    </h3>

    <div
      v-for="(review, index) in reviewList"
      :key="index"
      class="d-flex card bg-dark-subtle shadow p-3 flex-column mb-3"
    >
      <h4 class="fs-3 d-flex align-items-center fw-bold p-0 mb-2">
        {{ review.author }}
        <span
          class="bg-warning p-1 rounded bg-opacity-25 ms-2 user-select-none stars"
          >{{ review.rating }}</span
        >

        <span
          class="ms-auto text-muted created_at"
          v-html="setTimeAgo(review.created_at)"
        ></span>
      </h4>

      <div class="d-block">
        <span v-html="parseLinks(review.text)" class="fw-normal comment"></span>
      </div>
    </div>

    <div v-if="!reviewList.length" class="text-center mt-5">
      <h4 class="text-muted">No reviews yet</h4>
    </div>
  </div>
</template>

<style scoped>
.v-enter-active,
.v-leave-active {
  transition: opacity 1s;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.comment {
  color: #ccc;
  font-size: 15px;
  font-weight: 500;
  line-height: 1.4;
}

.stars {
  font-size: 14px;
}

.review__length {
  font-size: 16px !important;
}

.created_at {
  font-size: 12px;
}

img {
  height: 200px;
}
</style>
