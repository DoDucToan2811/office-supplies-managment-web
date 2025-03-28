<template>
  <div>
    <button v-if="showCreateSummaryButton" @click="createSummary" class="btn btn-info btn-fill float-right"
      style="position: absolute; top: 0; right: 0;min-width: fit-content; margin: 2%;">Tạo Phiếu Tổng Hợp</button>
    <table class="table">
      <thead>
        <slot name="columns">
          <tr style="background-color: rgb(220, 68, 5); color: white;">
            <th v-for="column in columns" :key="column" style="color: white;">{{ column }}</th>
            <th v-if="displayStatus" style="color: white;">Trạng thái</th>
            <th v-if="displayActions" style="color: white;">Thao tác</th>
          </tr>
        </slot>

      </thead>
      <tbody>
        <tr v-for="(item, rowIndex, column) in paginatedData" :key="rowIndex">

          <td>{{ (currentPage - 1) * pageSize + rowIndex + 1 }}</td>
          <td v-for="(column, colIndex) in columns.slice(1)" :key="colIndex">
            {{ itemValueByIndex(item, colIndex + 1) }}
          </td>

          <td v-if="displayStatus && (domain === 'request'|| domain==='summary')">
            <span :class="['status-badge', getStatusClass(item)]" style="min-width: 50px;">
              <i :class="getStatusIcon(item)"></i> {{ getStatusText(item) }}
            </span>
          </td>

          <td style="display: flex; justify-content: space-between; width: 50%;" v-if="displayActions">
            <button type="button" class="icon btn btn-info btn-sm" v-if="canView" @click="navigateToViewForm(item)">
              <i class="fa fa-eye"></i>
            </button>
            <button type="button" @click="navigateToEditForm(item)" class="icon btn btn-warning btn-sm"
              v-if="(item.Status === 'Chưa duyệt' && domain === 'request' && userRole === 'Employee') || domain === 'product' && userRole === 'Finance Management Employee'">
              <i class="fa fa-edit"></i>
            </button>
            <button type="button" @click="openDeleteDialog(item)" class="icon btn btn-danger btn-sm"
              v-if="(item.Status === 'Chưa duyệt' && domain === 'request' && userRole === 'Employee') || (domain === 'product' && userRole === 'Finance Management Employee')">
              <i class="fa fa-trash"></i>
            </button>
            <div v-if="showCheckboxColumn" class="icon btn-sm">
              <input type="checkbox" v-model="selectedRequests" :value="item.requestID"
                v-show="item.Status === 'Đã duyệt' && item.IsCollectedInSummary === false" class="icon"/>
            </div>
          </td>
        
        </tr>
      </tbody>
    </table>
    <nav>
      <ul class="pagination" style="color: rgb(220, 68, 5) !important;">
        <li class="page-item" :class="{ disabled: currentPage === 1 }" @click="changePage(currentPage - 1)">
          <button class="page-link">Previous</button>
        </li>
        <li v-for="page in totalPages" :key="page" class="page-item" :class="{ active: page === currentPage }"
          @click="changePage(page)">
          <button class="page-link">{{ page }}</button>
        </li>
        <li class="page-item" :class="{ disabled: currentPage === totalPages }" @click="changePage(currentPage + 1)">
          <button class="page-link">Next</button>
        </li>
      </ul>
    </nav>
    <div v-if="dialog" class="dialog-overlay">
      <div class="dialog-box">
        <p>Bạn có chắc chắn muốn xóa không?</p>
        <div class="dialog-actions">
          <button @click="dialog = false" class="btn btn-cancel">Hủy</button>
          <button @click="confirmDelete" class="btn btn-delete">Xóa</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: "l-table",
  props: {
    columns: Array,
    data: Array,
    apiURL: String,
    domain: String,
    pageSize: {
      type: Number,
      default: 10,
    },
    displayActions: Boolean,
    displayStatus: Boolean,
    canEdit: Boolean,
    canView: Boolean,
    canDelete: Boolean
  },
  data() {
    return {
      currentPage: 1,
      dialog: false,
      itemToDelete: null,
      type: ['success', 'danger', 'warning'],
      notifications: {
        topCenter: false
      },
      userRole: localStorage.getItem('userRole'),
      selectedRequests: [],
      token: localStorage.getItem('authToken'),
      userID: localStorage.getItem('userId'),
      productname: '',
    };
  },
  computed: {
    showCheckboxColumn() {
      return this.domain === 'request' && this.userRole === 'Finance Management Employee';
    },
    showCreateSummaryButton() {
      return this.selectedRequests.length > 0;
    },
    paginatedData() {
      const startIndex = (this.currentPage - 1) * this.pageSize;
      return this.data.slice(startIndex, startIndex + this.pageSize);
    },
    totalPages() {
      return Math.ceil(this.data.length / this.pageSize);
    },
  },
  methods: {
    itemValueByIndex(item, colIndex) {
      const values = Object.values(item);
      return values[colIndex] !== undefined ? values[colIndex] : '';
    },
    showCheckbox(item) {
      return this.domain === 'request' && this.userRole === 'Finance Management Employee' && item.Status === 'Đã duyệt';
    },
    itemValueByIndex(item, colIndex) {
      const values = Object.values(item);
      return values[colIndex] !== undefined ? values[colIndex] : '';
    },
    changePage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page;
      }
    },
    navigateToEditForm(item) {
      if (this.domain === 'product') {
        this.$router.push({ name: 'Edit Product', params: { id: item.productID } });
      } else if (this.domain === 'request') {
        this.$router.push({ name: 'Edit Request', params: { id: item.requestID } });
      }
    },
    navigateToViewForm(item) {
      if (this.domain === 'request') {
        this.$router.push({ name: 'View Request', params: { id: item.requestID, status: item.Status } });
      }else if (this.domain === 'summary'){
        this.$router.push ({ name: 'View Summary', params: { summaryId: item.summaryID } });
      }
    },
    openDeleteDialog(item) {
      this.itemToDelete = item;
      this.dialog = true;
    },
    async confirmDelete() {
  if (this.domain === 'product') {
    if (!this.itemToDelete) return;
    try {
      const productId = this.itemToDelete.productID;

      // Fetch the product details by its ID to get the product name
      const productResponse = await axios.get(`${this.apiURL}/${productId}`, {
        headers: { Authorization: `Bearer ${this.token}` }
      });
      const productName = productResponse.data.name;
      //console.log('Fetched product name:', productName);
      await axios.delete(`${this.apiURL}/${productId}`, {
        headers: { Authorization: `Bearer ${this.token}` }
      });

      const index = this.data.findIndex(product => product.productID === productId);
      if (index !== -1) {
        this.data.splice(index, 1);
      }

      // Fetch the username by userID
      const usernameResponse = await axios.get(`https://localhost:7162/User/getNameById${this.userID}`, {
        headers: { Authorization: `Bearer ${this.token}` }
      });
      const username = usernameResponse.data;
      //console.log('Fetched username:', username);

      // Fetch users with userTypeID == 2
      const usersType2Response = await axios.get('https://localhost:7162/User/users-by-type-id?userTypeID=2', {
        headers: { Authorization: `Bearer ${this.token}` },
      });
      const usersWithUserType2 = usersType2Response.data;

      // Fetch users with userTypeID == 4
      const usersType4Response = await axios.get('https://localhost:7162/User/users-by-type-id?userTypeID=4', {
        headers: { Authorization: `Bearer ${this.token}` },
      });
      const usersWithUserType4 = usersType4Response.data;

      // Combine users with userTypeID == 2 and userTypeID == 4
      const usersToNotify = [...usersWithUserType2, ...usersWithUserType4];

      // Send notifications to users with userTypeID == 2 and userTypeID == 4
      const notifications = usersToNotify.map(user => ({
        userID: user.userID,
        message: `The product ${productName} has been deleted by ${username}.`,
        requestID: productId,
        sender: this.userID,
      }));

      for (const notification of notifications) {
        //console.log(`Sending notification: ${JSON.stringify(notification)}`);
        await axios.post('https://localhost:7162/Notification', notification, {
          headers: { Authorization: `Bearer ${this.token}` },
        });
      }

      this.dialog = false;
      this.notifySuccess('top', 'right');

    } catch (error) {
      console.error("Delete error:", error);
      this.dialog = false;
      this.notifyError('top', 'right');
    }
  } else if (this.domain === 'request') {
    if (!this.itemToDelete) return;
    try {
      const requestId = this.itemToDelete.requestID;
      await axios.delete(`${this.apiURL}/${requestId}`, {
        headers: { Authorization: `Bearer ${this.token}` }
      });

      const index = this.data.findIndex(r => r.requestID === requestId);
      if (index !== -1) {
        this.data.splice(index, 1);
      }

      // Fetch the username by userID
      const usernameResponse = await axios.get(`https://localhost:7162/User/getNameById${this.userID}`, {
        headers: { Authorization: `Bearer ${this.token}` }
      });
      const username = usernameResponse.data;
      //console.log('Fetched username:', username);

      // Fetch the department leader
      const department = this.getUserDepartment(this.itemToDelete.userID);
      const departmentLeaderResponse = await axios.get(`https://localhost:7162/User/department-leader?department=${department}`, {
        headers: { Authorization: `Bearer ${this.token}` },
      });
      const departmentLeader = departmentLeaderResponse.data;

      // Send notifications to userID and department leader
      const notifications = [
        {
          userID: this.itemToDelete.userID,
          message: `Your request ${requestId} has been deleted by ${username}.`,
          requestID: requestId,
          sender: this.userID,
        },
        {
          userID: departmentLeader.userID,
          message: `Request ${requestId} from your department has been deleted by ${username}.`,
          requestID: requestId,
          sender: this.userID,
        }
      ];

      for (const notification of notifications) {
        //console.log(`Sending notification: ${JSON.stringify(notification)}`);
        await axios.post('https://localhost:7162/Notification', notification, {
          headers: { Authorization: `Bearer ${this.token}` },
        });
      }

      this.dialog = false;
      this.notifySuccess('top', 'right');

    } catch (error) {
      console.error("Delete error:", error);
      this.dialog = false;
      this.notifyError('top', 'right');
    }
  }
},
    async notifySuccess(verticalAlign, horizontalAlign) {
      this.$notifications.notify({
        message: `<span>Xóa sản phẩm thành công</span>`,
        icon: 'nc-icon nc-app',
        horizontalAlign: horizontalAlign,
        verticalAlign: verticalAlign,
        type: this.type[0]
      });
    },
    async notifyError(verticalAlign, horizontalAlign) {
      this.$notifications.notify({
        message: `<span>Xóa sản phẩm thất bại</span>`,
        icon: 'nc-icon nc-app',
        horizontalAlign: horizontalAlign,
        verticalAlign: verticalAlign,
        type: this.type[1]
      });
    },
    getStatusText(item) {
      //console.log("log trong table" + item.Status);
      return item.Status;
    },
    getStatusClass(item) {
      if (item.Status === "Đã duyệt") {
        return "status-approved";
      } else if (item.Status === "Đang xử lý") {
        return "status-pending";
      } else if (item.Status === "Không duyệt") {
        return "status-rejected";
      } else if (item.Status === "Chưa duyệt") {
        return "status-loading";
      }else if (item.Status === "Đã tổng hợp"){
        return "status-collected"
      }
    },
    getStatusIcon(item) {
      if (item.Status === "Đã duyệt") {
    return "fa fa-check-circle";
  } else if (item.Status === "Chưa duyệt") {
    return "fa fa-spinner";
  } else if (item.Status === "Đang xử lý") {
    return "fa fa-hourglass-half";
  } else if (item.Status === "Không duyệt") {
    return "fa fa-times-circle";
  } else if (item.Status === "Đã tổng hợp") {
    return "fa fa-archive"; // Icon cho trạng thái đã tổng hợp
  }
  return "";
    },
    createSummary() {
      this.$router.push({ name: 'Create Summary', params: {requestIds: this.selectedRequests.join(',')  } });
      //console.log("Tạo phiếu tổng hợp từ các request:", this.selectedRequests);
    },

  },

};
</script>

<style scoped>
input[type="checkbox"] {
  width: 100%; /* Kích thước bằng icon view */
  height: 100%; /* Kích thước bằng icon view */
  border: 2px solid #ccc; /* Viền */
  border-radius: 4px;
  cursor: pointer;
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  outline: none;
  position: relative;
}

input[type="checkbox"]:checked {
  background-color: rgb(220, 68, 5);
  border-color: rgb(220, 68, 5);
}

input[type="checkbox"]:checked::before {
  content: '\f00c';
  font-family: 'Font Awesome 5 Free';
  font-weight: 900;
  font-size: 14px; /* Điều chỉnh kích thước icon check nếu cần */
  color: white;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.status-badge {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  font-size: 14px;
  font-weight: bold;
  padding: 6px 12px;
  border-radius: 15px;
  text-align: center;
  width: max-content;
}

.status-approved {
  background-color: #28a745;
  color: white;
}

.status-pending {
  background-color: #ffc107;
  color: black;
}

.status-rejected {
  background-color: #dc3545;
  color: white;
}

.status-loading {
  background-color: #17a2b8;
  color: white;
}
.status-collected {
  background-color: #6c757d; /* Màu xám để thể hiện đã tổng hợp */
  color: white;
  font-weight: bold;
}

.status-badge i {
  font-size: 16px;
}

.status-approved {
  color: white;
  font-weight: bold;
}

.status-pending {
  color: white;
  font-weight: bold;
}

tbody tr {
  transition: box-shadow 0.2s ease-in-out;
}

td {
  border: none !important;
}

tbody tr:hover {
  box-shadow: 0 5px 12px rgba(0, 0, 0, 0.2);
  background-color: inherit;
}


.table-striped tbody tr:nth-of-type(odd) {
  background-color: rgba(248, 153, 113, 0.2);
  z-index: 1;
}

.btn {
  width: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.btn-danger {
  border: 2px solid #dc3545 !important;
}

.btn-warning {
  border: 2px solid #e0a800 !important;
  /* Màu đậm hơn để dễ thấy */
}

.icon {
  margin-right: 20px;
  cursor: pointer;
}

.icon:hover {
  color: orangered;
}

.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
}

.pagination .page-item.disabled .page-link {
  pointer-events: none;
  opacity: 0.5;
  color: rgb(220, 68, 5);
}

.pagination .page-item.active .page-link {
  background-color: rgb(220, 68, 5);
  border-color: rgb(220, 68, 5);
  color: #fff;
}

.pagination .page-item .page-link {
  color: rgb(220, 68, 5);
  /* Màu cam cho tất cả các số trang */
}


/* CSS cho Dialog */
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}

.dialog-box {
  background: white;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
  width: 400px;
}

.dialog-actions {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

.btn {
  padding: 8px 16px;
  /* border: none; */
  cursor: pointer;
}

.btn-cancel {
  background: gray;
  color: white;
}

.btn-delete {
  background: red;
  color: white;
}
</style>
