<template>
  <header class="main-header">
    <nav class="nav-menu">
      <div class="nav-left">
        <img src="../../public/Logo1.svg" class="logo" @click="movetohomepage">
        <img src="../assets/bumpword.svg" alt="Bump" class="bumpword" @click="movetohomepage">
        <a @click="movetomyprofile" class="profile-link">My Profile</a>
      </div>
      <div class="nav-center">
        <div ref="searchContainer" class="search-container" :class="{ searched: searchResults.length }">
          <input type="text" name="search" placeholder="Search" class="search" v-model="searchQuery" @input="searchUsers"
            @focus="isSearchFocused = true" @blur="handleSearchBlur">
          <img src="../assets/search.svg" alt="Search" class="search-icon">
          <ul v-if="searchResults.length" :class="['dropdown', { 'expanded': isSearchFocused }]">
            <li v-for=" result  in  searchResults " :key="result._id" @click="movetoprofile(result._id)">
              <img class="searchimg" :src="result.Picture" :class="{ me: (result._id === userId) }"
                alt="Profile Picture" />
              <p>{{ result.GamerTag }}</p>
            </li>
          </ul>
        </div>
      </div>
      <div class="nav-right">
        <button class="btn" @click="movetomatching">Let's Bump</button>

        <div class="notidiv">
          <img v-if="notifications === false || notifications.length === 0" class="notiimg" src="../assets/noti_off.svg"
            @click="toggleDropdown">
          <img v-else class="notiimg" src="../assets/closed.svg" @blur="isDropNotiActive = !isDropNotiActive"
            @click="toggleDropdown">
          <div class="dropdownnoti" :class="{ active: isDropNotiActive }">
            <div class="triangle"></div>
            <div class="triangle1"></div>
            <h3>Notifications</h3>
            <ul>
              <li v-for="notification in notifications" :key="notification">
                <img class="notipicture" :src="notification.Picture" alt="Profile Picture" />
                <div class="bumpgamertagnoti">
                  <h2 class="h2noti">New Bump</h2>
                  <p class="gamertagnoti">By {{ notification.GamerTag }}</p>
                </div>
                <div class="discordandcopydiv" @mouseenter="notification.enablecopy = true"
                  @mouseleave="notification.enablecopy = false">
                  <p class="discordnoti">
                    {{ notification.Discord }}
                  </p>
                  <img v-if="notification.enablecopy" @click="copyText(notification.Discord)" class="copy"
                    src="../assets/content_copy.svg" />
                </div>
                <p class="timenoti">{{ date(notification.updatedAt) }}</p>
              </li>
            </ul>
          </div>
        </div>
        <img class="logout" src="../assets/logout.svg" alt="Logout" @click="logout">
      </div>
    </nav>
  </header>
</template>
  
<script>
import axios from 'axios';
import loading from './loading.vue'
import { debounce } from 'lodash';

export default {
  name: "navbar",
  components: {
    loading
  },
  props: ['reload'],
  data() {
    return {
      searchQuery: "",
      searchResults: [],
      notifications: [],
      isDropNotiActive: false,
      userId: this.$route.query.id,
      isloading: false,
      clickOutsideListener: null,
      defaultProfilePicture: 'https://res.cloudinary.com/dk9nwmeth/image/upload/v1619629599/ProfilePictures/default_profile_picture.png',
      closeResultsTimeout: null,
      isSearchFocused: false,
      enablecopy: false,
      inactivityTimeout: null,
      reload: false,
    };
  },
  mounted() {
    this.resetInactivityTimeout();
    window.addEventListener('mousemove', this.resetInactivityTimeout);
    window.addEventListener('keydown', this.resetInactivityTimeout);
  },
  beforeDestroy() {
    window.removeEventListener('mousemove', this.resetInactivityTimeout);
    window.removeEventListener('keydown', this.resetInactivityTimeout);
  },
  created() {
    this.getNotifications();
  },
  methods: {
    resetInactivityTimeout() {
      clearTimeout(this.inactivityTimeout);
      this.inactivityTimeout = setTimeout(this.logout, 30 * 60 * 1000);
    },
    handleSearchBlur() {
      this.isSearchFocused = false;
      this.cancelCloseSearchResults();
    },
    closeSearchResults() {
      this.closeResultsTimeout = setTimeout(() => {
        this.searchResults = [];
      }, 200);
    },
    cancelCloseSearchResults() {
      if (this.closeResultsTimeout) {
        clearTimeout(this.closeResultsTimeout);
      }
    },
    searchUsers: debounce(async function () {
      if (this.searchQuery.trim() === "") {
        this.searchResults = [];
        return;
      } else {
        try {
          console.log(this.searchQuery);
          this.searchResults = [];
          var addr = 'https://backend-project-vzn7.onrender.com/search';
          const response = await axios.post(addr, {
            "searchQuery": this.searchQuery.trim(),
            "userId": this.userId,
          })
          this.searchResults = response.data
        } catch (error) {
          console.error(error);
        }
      }
    }, 300),
    toggleDropdown() {
      this.isDropNotiActive = !this.isDropNotiActive;
      if (this.isDropNotiActive) {
        this.getNotifications();
      }
    },
    async logout() {
      try {
        if (this.reload) return;
        this.$router.push('/login_signup')

        this.isloading = true;
        var addr = 'https://backend-project-vzn7.onrender.com/logout/' + this.userId;
        console.log('logout:' + addr);
        const response = await axios.get(addr);

        var res = response.data;
        console.log("logout:" + res);

      } catch (error) {
        console.error(error);
        console.log(error);
      }
      this.isloading = false;
      this.reload = false;
    },
    movetomyprofile() {
      console.log('before push' + this.userId + ' dif ' + this.$route.query.id)
      this.$router.push({ name: 'profile', query: { id: this.userId }, params: { differentUserId: this.userId } });

      this.$emit('openProfile', this.userId)
    },
    movetomatching() {
      if (this.$route.name != 'matching') {
        this.$router.push({ name: 'matching', query: { id: this.userId } });
      }
      else {
        this.reload = true;
        window.location.reload();

      }
    },
    movetohomepage() {
      this.$router.push({ name: 'homepage', query: { id: this.userId } });
    },
    movetoprofile(id) {
      this.searchResults = [];
      this.$router.push({ name: 'profile', query: { id: this.userId }, params: { differentUserId: id } });
      this.$emit('openProfile', id)
    },
    handleClickOutside(event) {
      const isClickedInsideSearchContainer = this.$refs.searchContainer && this.$refs.searchContainer.contains(event.target);
      const isClickedInsideDropdown = event.target.closest(".dropdown");
      if (!isClickedInsideSearchContainer && !isClickedInsideDropdown) {
        this.searchResults = [];
      } else {
        this.cancelCloseSearchResults();
      }
    },
    copyText(text) {
      navigator.clipboard.writeText(text);
    },
    async getNotifications() {
      var addr = `https://backend-project-vzn7.onrender.com/getnotifications/${this.userId}`;
      console.log('getNotifications:' + addr)
      try {
        const response = await axios.get(addr);
        this.notifications = response.data;
        console.log(this.notifications)
      } catch (error) {
        console.error('Error occurred during the GET request:', error);
      }
    },
    date(date) {
      var date = new Date(date);
      const options = {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit',
        timeZone: 'Asia/Jerusalem',
      };
      return date.toLocaleString('en-IL', options);
    }
  }
};

</script>
  
<style scoped>
.main-header {
  background-color: var(--pagebgcolor);
  color: var(--white);
  display: flex;
  justify-content: center;
  align-items: center;
  height: 80px;
  width: 100%;
  border-bottom: 2px solid var(--stroke);
  position: fixed;
  top: 0;
  left: 0;
  z-index: 999;
}

.nav-menu {
  display: flex;
  width: 100%;
  padding: 0 30px;
}

.nav-left,
.nav-right {
  display: flex;
  align-items: center;
}

.nav-left {
  flex: 1;
}

.nav-right {
  flex: 1;
  justify-content: flex-end;
}

.logo {
  display: flex;
  height: 55px;
  width: 55px;
  margin-right: 20px;
}

.logo:hover {
  cursor: pointer;
}

.nav-center {
  flex: 1;
  display: flex;
  justify-content: center;
}

.search-container {
  position: relative;
  display: flex;
  align-items: center;
}

.search-container.searched .search {
  border-bottom-left-radius: 0;
  border-bottom-right-radius: 0;
  border-bottom: none;
}

.search-container.searched .dropdown {
  display: block;
}

.search {
  border: none;
  background-color: var(--thirdcolor);
  border-radius: 15px;
  color: var(--white);
  border: 2px solid var(--stroke);
  font-family: var(--mainfont);
  font-size: 14px;
  font-weight: 400;
  line-height: 1.5;
  letter-spacing: 0.5px;
  width: 200px;
  height: 40px;
  transition: all 0.2s ease;
}

.searchimg {
  width: 45px;
  height: 45px;
  border-radius: 50%;
  border: 2px solid var(--white);
  margin-right: 20px;
  margin-left: 10px;
}

.search:focus {
  outline: none;
  border-color: var(--white);
  width: 250px;
}

.search::placeholder {
  color: #b9b9b9;
  font-family: var(--mainfont);
  font-size: 14px;
  font-weight: 400;
  line-height: 1.5;
  letter-spacing: 0.5px;
}

.dropdown {
  position: absolute;
  display: none;
  left: 0;
  width: 215px;
  background-color: var(--thirdcolor);
  border-radius: 0 0 15px 15px;
  list-style: none;
  border: 2px solid var(--stroke);
  border-top: none;
  top: 48px;
  margin-top: 0;
  z-index: 1;
  margin: 0;
  padding-left: 0;
  max-height: 270px;
  overflow-y: auto;
  transition: all 0.2s ease;

}

.dropdown.expanded {
  width: 265px;
  border: 2px solid var(--white);
  border-top: none;
}

.dropdown li {
  color: var(--white);
  cursor: pointer;
  padding: 5px;
  display: flex;
  align-items: center;
}

.dropdown li:last-child {
  border-radius: 0 0 15px 15px;
}

.dropdown li:hover {
  background-color: var(--hover);
  cursor: pointer;
}

.me {
  border: 2px solid var(--main);
}


.search::-webkit-input-placeholder,
.search:focus::-webkit-input-placeholder,
.search::-moz-placeholder,
.search:focus::-moz-placeholder,
.search:-moz-placeholder,
.search:focus:-moz-placeholder,
.search:-ms-input-placeholder,
.search:focus:-ms-input-placeholder {
  color: var(--white);
  opacity: 0.6;
}

.search-icon {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  right: 10px;
  width: 20px;
  height: 20px;
  transition: all 0.3s ease;
}


.search:focus+.search-icon {
  transform: translateY(-50%) rotate(90deg);
}


.profile-link {
  color: var(--white);
  text-decoration: none;
  margin-right: 20px;
  transition: 0.5s;
}

.profile-link:hover {
  text-decoration: underline;
  text-decoration-color: var(--main);
  cursor: pointer;

}

.logout {
  height: 30px;
  width: 30px;
}

.logout:hover {
  cursor: pointer;
}

.bumpword {
  margin-right: 20px;
  margin-left: -20px;
  height: 30px;
  width: 80px;
}

.bumpword:hover {
  cursor: pointer;
}

.btn {
  margin-right: 40px;
  padding: 8px 20px;
}


.notidiv {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 20px;
}

.notiimg {
  margin-right: 20px;
  height: 30px;
  width: 30px;
  cursor: pointer;
}

.dropdownnoti {
  display: none;
  position: absolute;
  top: 100%;
  margin-top: 15px;
  background-color: var(--pagebgcolor);
  width: 350px;
  border-radius: 5px;
  border: 3px solid var(--stroke);
  margin-right: 333px;
  padding-top: 10px;
  opacity: 0;
  animation-name: fadeIn;
  animation-duration: 0.3s;
  animation-fill-mode: forwards;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}


.dropdownnoti.active {
  display: block;
}

.dropdownnoti .triangle {
  position: absolute;
  top: -15px;
  right: 5px;
  width: 26px;
  height: 16px;
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
  background-color: var(--stroke);
  z-index: -1;
}

.dropdownnoti .triangle1 {
  position: absolute;
  top: -12px;
  right: 8px;
  width: 20px;
  height: 15px;
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
  background-color: var(--pagebgcolor);
}

.dropdownnoti h3 {
  margin: 0;
  padding: 0;
  color: var(--white);
  font-family: var(--mainfont);
  font-weight: 400;
  line-height: 1.5;
  letter-spacing: 0.5px;
  margin-left: 10px;
  padding-bottom: 5px;
  font-size: 20px;
}

.dropdownnoti ul {
  list-style: none;
  padding: 0;
  margin: 0;
  max-height: 310px;
  overflow-y: auto;
}

.dropdownnoti li {
  position: relative;
  color: var(--white);
  padding: 5px;
  display: flex;
  gap: 10px;
  align-items: center;
}

.timenoti {
  position: absolute;
  bottom: 5px;
  right: 5px;
  color: var(--grey);
  font-family: var(--mainfont);
  font-weight: 300;
  font-size: 9px;
}

.dropdownnoti li:hover {
  background-color: var(--hover);
}

.h2noti {
  margin: 0;
  padding: 0;
  color: var(--white);
  font-family: var(--mainfont);
  font-weight: 400;
  font-size: 20px;
}

.gamertagnoti {
  margin: 0;
  padding: 0;
  color: var(--grey);
  font-family: var(--mainfont);
  font-weight: 300;
  max-width: 107.39px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.discordnoti {
  margin: 0;
  padding: 0;
  color: var(--white);
  font-family: var(--mainfont);
  font-weight: 400;
  max-width: 120px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.discordnoti {
  display: flex;
  align-items: center;
  color: var(--grey);
  transition: 0.3s ease, color 0.3s ease;
  cursor: default;
  padding: 0;
}

.discordandcopydiv {
  display: flex;
  flex-direction: row;
  gap: 5px;
  align-items: center;
  margin-left: 10px;
}

.discordnoti:hover {
  color: var(--white);
}


.bumpgamertagnoti {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  white-space: nowrap;
}

.notipicture {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  border: 2px solid var(--white);
}

.copy {
  width: 15px;
  height: 15px;
  cursor: pointer;
  filter: grayscale(100%);
  transition: transform 0.3s ease;
}

.copy:hover {
  transform: scale(1.1);

}

.copy:active {
  transform: scale(calc(0.1));
}
</style>
  