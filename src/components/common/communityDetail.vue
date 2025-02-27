<template>
  <div class="feedDetail-container">
    <div class="container">
      <div class="photo-section">
        <img :src="feed.imageUrl" class="feed-img" />
      </div>
      <div class="content-section">
        <div class="user-header">
          <div class="user-info">
            <img :src="feed.userimg" alt="profile" class="photo" />
            <div>{{ feed.username }}</div>
          </div>
        </div>
        <div class="middle-container">
          <div class="time-container">
            작성일자: {{ new Date(feed.time).toLocaleString() }}
          </div>
        </div>
        <div class="content mb-10">
          <p>{{ feed.content }}</p>
        </div>
        <div class="comments mb-10">
          <div>Comments</div>
          <ul>
            <li v-for="(comment, index) in feed.comments" :key="index">
              {{ comment }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

// 반응형 feedData 객체 생성
const feedData = ref({
  username: '',
  userimg: '',
  content: '',
  time: '',
  imageUrl: '',
  comments: []
});

// props로 feed를 받아옴
const props = defineProps({
  feed: {
    type: Object,
    required: true
  }
});

// setFeed 함수: props로 받아온 값을 feedData에 설정
const setFeed = () => {
  feedData.value = {
    username: props.feed.username,
    userimg: props.feed.userimg,
    content: props.feed.content,
    time: props.feed.time,
    imageUrl: props.feed.imageUrl,
    comments: props.feed.comments
  };
};

// 컴포넌트가 마운트될 때 setFeed 함수 실행
onMounted(() => {
  setFeed();
});
</script>

<style scoped>

.feedDetail-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 95%;
  height: 100dvh;
}

.container {
  display: flex;
  flex-direction: row;
  width: 95%;
  height: 800px;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0;
  border: 5px solid #a68b6a;
  border-radius: 10px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
}

.photo-section {
  flex: 1.8;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.feed-img {
  width: 100%;
  height: 100%;
  border-radius: 5px;
  object-fit: cover;
}

.photo {
  height: 30px;
  border-radius: 50%; /* 원형 이미지로 표시 */
  margin-right: 10px;
}

.content-section {
  flex: 1;
  height: 98%;
  display: flex;
  flex-direction: column;
  padding: 15px;
  overflow: hidden;
  background-color: #fff;
  border-radius: 8px;
  border: none;
}

.user-info {
  display: flex;
  flex-direction: row;
  align-items: center;
  flex: 1;
}

.user-info div {
  font-family: "GothicA1-Light";
  font-size: 1em;
  color: #000;
}

.content {
  flex: 3;
  overflow-wrap: break-word; /* 긴 단어나 URL이 박스를 넘어가지 않도록 줄바꿈 */
  word-wrap: break-word; /* IE11 이전 호환 */
  word-break: break-word; /* 긴 단어들이 자동으로 잘리도록 */
  overflow: hidden; /* 넘치는 텍스트 숨기기 */
}

.content p {
  font-family: "GothicA1-Light";
  margin-top: 10px;
  font-size: 0.85em;
  line-height: 1.5;
}

.comments {
  flex: 2;
}

.comments div {
  font-family: "ghanachoco";
  margin-bottom: 10px;
  font-size: 1em;
  color: #76b852; /* fallback for old browsers */
  color: -webkit-linear-gradient(
    to right,
    #8dc26f,
    #76b852
  ); /* Chrome 10-25, Safari 5.1-6 */
  color: linear-gradient(
    to right,
    #8dc26f,
    #76b852
  ); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
}

.comments ul {
  height: 95%;
  list-style-type: none;
  border: none;
  border-radius: 10px;
  background-color: #f5f5f5;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
  padding: 5px;
}

.comments li {
  font-family: "GothicA1-Light";
  margin-top: 10px;
  font-size: 0.9em;
}

.middle-container {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  margin-top: 10px;
}

.time-container {
  font-family: "GothicA1-Light";
  font-size: 0.8em;
  line-height: 0.8em;
  color: #999;
  width: 100%;
  padding: 5px;
}

.btn-container {
  display: flex;
  flex-direction: row;
  width: 100%;
  justify-content: right;
  flex: 0.2;
}

.modify-button-section {}

.modify-button {
  background: #76b852; /* fallback for old browsers */
  background: -webkit-linear-gradient(
    to right,
    #8dc26f,
    #76b852
  ); /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(
    to right,
    #8dc26f,
    #76b852
  ); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
  color: white;
  padding: 2px 6px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
  margin-left: 10px;
}

.material-icons {
  font-size: 21px;
  color: #fff;
}
</style>
