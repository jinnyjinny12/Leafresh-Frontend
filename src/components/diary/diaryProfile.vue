<template>
  <div class="profile-container">
    <!-- 프로필 이미지와 사용자 정보 -->
    <div class="profile-header" v-if="profileExists">
      <div class="profile-image-container">
        <img :src="profileImage" alt="Profile Image" class="profile-image" />
      </div>
      <div class="profile-info">
        <div class="profile-header-row">
          <!-- 유저 닉네임을 표시하는 부분 -->
          <h2 class="profile-username">{{ userNickname }}</h2>

           <!-- 팔로잉, 프로필 수정, 피드 추가, 식물 등록 버튼 표시 -->
           <button
            class="edit-button"
            v-if="!isCurrentUserProfile"
            @click="toggleFollow"
            :title="isFollowing ? '팔로잉 취소' : '팔로잉'"
          >
            <span class="material-icons">{{ isFollowing ? 'person_remove' : 'person_add' }}</span>
            {{ isFollowing ? '팔로잉 취소' : '팔로잉' }}
          </button>

          <button
            class="edit-button"
            v-if="isCurrentUserProfile"
            @click="openEditModal"
            title="프로필 수정"
          >
            <span class="material-icons">manage_accounts</span>
          </button>

          <button
            class="edit-button"
            v-if="isCurrentUserProfile"
            @click="goToFeedAdd"
            title="피드 추가"
          >
            <span class="material-icons">edit_note</span>
          </button>

          <button
            class="edit-button"
            v-if="isCurrentUserProfile"
            @click="openPlantAddModal"
            title="식물 등록"
          >
            <span class="material-icons">add_circle_outline</span>
          </button>
        </div>

        <div class="profile-stats">
          <span>반려식물 {{ followerPlants }}개</span>
          <span>판매식물 {{ salePlants }}개</span>
          <span>팔로워 {{ followers }}명</span>
        </div>
        <div class="profile-bio">
          <p class="profile-title">{{ profileTitle }}</p>
          <p class="profile-description">{{ profileDescription }}</p>
        </div>
      </div>
    </div>

    <!-- 프로필 수정 모달 컴포넌트 -->
    <EditProfileModal
      :isOpen="isEditModalOpen"
      :closeModal="closeEditModal"
      :profileTitle="profileTitle"
      :profileDescription="profileDescription"
      :currentUsername="userNickname"
    />

    <!-- 식물 등록 모달 컴포넌트 -->
    <PlantAddModal
      :isOpen="isPlantAddModalOpen"
      :closeModal="closePlantAddModal"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, watchEffect, watch, computed } from 'vue';
import axios from 'axios';
import { useRouter, useRoute } from 'vue-router';
import EditProfileModal from './EditProfileModal.vue';
import PlantAddModal from '@/views/PlantAddModal.vue';
import { useUserstore } from '@/stores/users'; // 사용자 스토어 불러오기

const userStore = useUserstore();
const route = useRoute();
const router = useRouter();
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;

const profileImage = ref('');
const userNickname = ref(route.params.userNickname);
const profileTitle = ref('');
const profileDescription = ref('');
const followerPlants = ref(0);
const salePlants = ref(0);
const followers = ref('');

// 모달 상태
const isEditModalOpen = ref(false);
const isPlantAddModalOpen = ref(false);

// 추가된 변수들
const profileExists = ref(false); // 프로필 존재 여부 추가
const isFollowing = ref(false); // 팔로잉 상태를 관리하는 변수 추가
const localToken = localStorage.getItem("accessToken");

// 현재 사용자와 프로필 사용자가 같은지 확인하는 계산 속성
const isCurrentUserProfile = computed(() => {
  return userStore.userNickname === userNickname.value;
});

// 프로필 수정 모달 열기
const openEditModal = () => {
  isEditModalOpen.value = true;
};

// 프로필 수정 모달 닫기
const closeEditModal = () => {
  isEditModalOpen.value = false;
};

// 식물 등록 모달 열기
const openPlantAddModal = () => {
  isPlantAddModalOpen.value = true;
};

// 식물 등록 모달 닫기
const closePlantAddModal = () => {
  isPlantAddModalOpen.value = false;
};

// 팔로잉 상태 확인
const checkFollowingStatus = async () => {
  try {
    const response = await axios.get(`${API_BASE_URL}/follow/status`, {
      params: { followingNickname: userNickname.value },
      headers: {
        Authorization: `Bearer ${localToken}`,
      },
    });

    // API 응답 확인
    console.log("API 응답 데이터:", response.data);

    // 응답 데이터가 예상한 형태인지 확인하고 수정된 필드명을 사용
    if (response && response.data && typeof response.data.following === 'boolean') {
      isFollowing.value = response.data.following; // 올바른 필드명을 사용하여 팔로잉 상태 업데이트
      console.log("팔로잉 상태 업데이트:", isFollowing.value);
    } else {
      console.error('API 응답 형식이 올바르지 않습니다:', response.data);
    }
  } catch (error) {
    console.error('팔로잉 상태를 확인하는 중 오류 발생:', error);
  }
};


console.log("팔로잉 상태 : "+ isFollowing.value);

// 팔로잉/언팔로우 토글
const toggleFollow = async () => {
  try {
    if (isFollowing.value) {
      await axios.delete(`${API_BASE_URL}/follow`, {
        data: { followingNickname: userNickname.value },
        headers: {
          Authorization: `Bearer ${localToken}`,
        },
      });
      isFollowing.value = false;
      followers.value -= 1; // 팔로워 수 감소
    } else {
      await axios.post(`${API_BASE_URL}/follow`, {
        followingNickname: userNickname.value,
      }, {
        headers: {
          Authorization: `Bearer ${localToken}`,
        },
      });
      isFollowing.value = true;
      followers.value += 1; // 팔로워 수 증가
    }
  } catch (error) {
    console.error('팔로우/언팔로우 중 오류 발생:', error);
  }
};


// 사용자 프로필 정보 가져오기
const fetchUserProfileDetails = async () => {
  try {
    const nickname = route.params.userNickname;

    // 사용자 정보 API 호출
    const userResponse = await axios.get(`${API_BASE_URL}/user/info-by-nickname`, {
      params: { nickname: nickname },
      headers: {
        Authorization: `Bearer ${localToken}`,
      },
    });

    if (typeof userResponse.data !== 'object') {
      throw new Error('Invalid JSON response');
    }

    followers.value = userResponse.data.followers;

    // 팔로워 수 API 호출 (추가된 부분)
    const followersResponse = await axios.get(`${API_BASE_URL}/follow/followers/count`, {
      params: { nickname: nickname },
      headers: {
        Authorization: `Bearer ${localToken}`,
      },
    });

    // 팔로워 수를 상태에 반영
    if (typeof followersResponse.data === 'number') {
      followers.value = followersResponse.data; // API로부터 받은 팔로워 수로 업데이트
      console.log('팔로워 수:', followers.value);
    } else {
      console.error('팔로워 수 API 응답 형식이 올바르지 않습니다:', followersResponse.data);
    }

    // 이미지 경로를 FTP 서버에서 가져옴
    const imageResponse = await axios.get(`${API_BASE_URL}/ftp/image`, {
      params: { path: userResponse.data.imageUrl },
      headers: {
        Authorization: `Bearer ${localToken}`,
      },
      responseType: 'blob',
    });

    profileImage.value = URL.createObjectURL(imageResponse.data);

    const profileResponse = await axios.get(`${API_BASE_URL}/profile/info-by-nickname`, {
      params: { nickname: nickname },
      headers: {
        Authorization: `Bearer ${localToken}`,
      },
    });

    if (typeof profileResponse.data !== 'object') {
      throw new Error('Invalid JSON response');
    }

    profileTitle.value = profileResponse.data.profileTitle;
    profileDescription.value = profileResponse.data.profileDescription;
    profileExists.value = true; // 프로필 존재 여부 설정

    // 팔로잉 상태 확인
    if (!isCurrentUserProfile.value) {
      checkFollowingStatus();
    }

  } catch (error) {
    console.error('Error fetching user profile details:', error);
    profileExists.value = false; // 오류 시 프로필 존재 여부 설정
  }
};


// 닉네임 변경을 감시하고 프로필을 다시 가져오는 watch 설정
watch(
  () => route.params.userNickname,
  (newNickname) => {
    userNickname.value = newNickname; // 닉네임 업데이트
  }
);

// 데이터 변경 감시 및 반영
watchEffect(() => {
  if (localToken) {
    console.log('프로필 정보를 새로 가져옵니다.');
    fetchUserProfileDetails();
    checkFollowingStatus(); // 로그아웃 후 다시 로그인 시에도 팔로잉 상태를 확인하도록 추가
  }
});

// 컴포넌트 마운트 시 프로필 정보 가져오기
onMounted(async () => {
  if (localToken) {
    // 사용자가 로그인된 상태이면 프로필 정보 가져오기
    fetchUserProfileDetails();
  } else {
    console.warn('사용자 토큰이 존재하지 않습니다.');
  }
});
</script>


<style scoped>
/* CSS는 동일하게 유지합니다. */
.profile-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30px;
  max-width: 1200px;
  margin: 0 auto;
  border-radius: 12px;
}

.profile-header {
  display: flex;
  align-items: center;
}

.profile-image-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.profile-image {
  width: 200px;
  height: 200px;
  border-radius: 50%;
  background-color: #ddd;
}

.profile-info {
  padding-left: 30px;
}

.profile-header-row {
  display: flex;
  align-items: center;
  margin-bottom: 15px;
  padding-bottom: 10px;
}

.profile-username {
  font-size: 32px;
  font-weight: bold;
  margin-right: 10px;
}

.follow-button {
  padding: 8px 15px;
  margin-left: 30px;
  border: none;
  background-color: #e0e0e0;
  border-radius: 8px;
  cursor: pointer;
  font-size: 18px;
}

.follow-button:hover {
  background-color: #d4d4d4;
}

.profile-stats {
  margin: 15px 0;
  font-size: 20px;
  display: flex;
  padding-bottom: 20px;
  gap: 30px;
}

.profile-bio {
  margin-top: 15px;
  font-size: 18px;
}

.profile-title {
  font-weight: bold;
  font-size: 20px;
  padding-bottom: 10px;
}

.profile-description {
  white-space: pre-line;
  color: #666;
  font-size: 18px;
}

.edit-button {
  margin-left: 10px;
  padding: 5px 10px;
  background-color: #1ab546;
  border: none;
  color: white;
  cursor: pointer;
  border-radius: 5px;
}

.edit-button:hover {
  background-color: #148838;
}
</style>
