<template>
  <nav class="navbar navbar-expand-lg">
    <div class="container-fluid">
      <a class="navbar-brand" href="#">QUẢN LÝ CẤP PHÁT VĂN PHÒNG PHẨM</a>
      <button type="button"
              class="navbar-toggler navbar-toggler-right"
              :class="{ toggled: $sidebar.showSidebar }"
              aria-controls="navigation-index"
              aria-expanded="false"
              aria-label="Toggle navigation"
              @click="toggleSidebar">
        <span class="navbar-toggler-bar burger-lines"></span>
        <span class="navbar-toggler-bar burger-lines"></span>
        <span class="navbar-toggler-bar burger-lines"></span>
      </button>
      <div class="collapse navbar-collapse justify-content-end">
        <ul class="navbar-nav ml-auto">
          <li class="nav-item dropdown" ref="dropdown" @click="logout" style="cursor: pointer;">
              <i class="fa fa-sign-out-alt"></i> 
              <span class="ml-2">Đăng xuất</span>
              <!-- <i class="fa fa-chevron-down ml-2 logout" @click="logout()"></i>  -->
          </li>
        </ul>
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  data() {
    return {
      userName: localStorage.getItem("userName") || "Người dùng",
      isDropdownOpen: false
    };
  },
  methods: {
    toggleSidebar() {
      this.$sidebar.displaySidebar(!this.$sidebar.showSidebar);
    },
    toggleDropdown(event) {
      this.isDropdownOpen = !this.isDropdownOpen;
      event.stopPropagation(); // Prevents the document click from closing immediately
    },
    closeDropdown(event) {
      if (this.$refs.dropdown && !this.$refs.dropdown.contains(event.target)) {
        this.isDropdownOpen = false;
      }
    },
    viewProfile() {
      this.$router.push("/profile"); // Change this to your actual profile route
    },
    logout() {
      localStorage.clear();
      this.$router.push("/");
    }
  },
  mounted() {
    document.addEventListener("click", this.closeDropdown);
  },
  beforeUnmount() {
    document.removeEventListener("click", this.closeDropdown);
  }
};
</script>

<style scoped>
/* .nav-item {
  position: relative;
  cursor: pointer;
} */

/* Dropdown menu positioning */
/* .dropdown-menu {
  display: none;
  position: absolute;
  top: 100%;
  right: 0;
  min-width: 150px;
  background-color: white;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
  border-radius: 5px;
  padding: 8px 0;
  z-index: 1000;
}

.dropdown-menu.show {
  display: block;
}

.dropdown-item {
  padding: 8px 16px;
  cursor: pointer;
  display: flex;
  align-items: center;
}

.dropdown-item i {
  margin-right: 10px;
}

.dropdown-item:hover {
  background-color: #f8f9fa;
} */

/* .nav-link {
  display: flex;
  align-items: center;
} */

/* Ensure the dropdown arrow is visible */
/* .dropdown-arrow {
  font-size: 14px;
  transition: transform 0.2s;
  color: #333; 
} */

/* .dropdown-arrow.open {
  transform: rotate(180deg);
} */
.nav-item:hover {
color: red;
cursor: pointer;
}
</style>
