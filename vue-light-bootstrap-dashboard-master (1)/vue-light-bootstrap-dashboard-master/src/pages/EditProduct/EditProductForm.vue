<template>
  <card>
    <h4 slot="header" class="card-title">Chỉnh sửa sản phẩm</h4>
    <form>
      <div class="row">
        <div class="col-md-6">
          <label for="productname">Tên sản phẩm</label>
          <base-input 
            type="text" 
            placeholder="Nhập tên sản phẩm" 
            name="productname" 
            id="productname"
            v-model="product.Name">
          </base-input>
        </div>
        <div class="col-md-6">
          <label for="productcode">Mã sản phẩm</label>
          <base-input 
            type="text" 
            name="productcode" 
            id="productcode" 
            placeholder="Nhập mã sản phẩm"
            v-model="product.Code">
          </base-input>
        </div>
      </div>

      <div class="row">
        <div class="col-md-6">
          <label for="productcurrency">Đơn vị tính</label>
          <select class="form-control" v-model="product.UnitCurrency">
            <option value="" disabled>Chọn đơn vị tính</option>
            <option v-for="currency in unitCurrencies" :key="currency" :value="currency">
              {{ currency }}
            </option>
          </select>
        </div>
        <div class="col-md-6">
          <label for="productprice">Đơn giá</label>
          <BaseInput 
            type="number" 
            placeholder="Nhập đơn giá" 
            name="productprice" 
            id="productprice"
            v-model="product.UnitPrice">
          </BaseInput>
        </div>
      </div>

      <div class="row">
        <div class="col-md-6">
          <label for="createdby">Người tạo</label>
          <BaseInput 
            type="text" 
            name="createdby" 
            id="createdby" 
            v-model="createdByName"
            disabled>
          </BaseInput>
        </div>
        <div class="col-md-6">
          <label for="createddate">Ngày tạo</label>
          <BaseInput 
            type="text" 
            name="createddate" 
            id="createddate" 
            v-model="product.CreatedDate"
            disabled>
          </BaseInput>
        </div>
      </div>

      <div class="row">
        <div class="col-md-6">
          <label for="adjustedby">Người chỉnh sửa</label>
          <BaseInput 
            type="text" 
            name="adjustedby" 
            id="adjustedby" 
            v-model="adjustedByName"
            disabled>
          </BaseInput>
        </div>
        <div class="col-md-6">
          <label for="adjustdate">Ngày chỉnh sửa</label>
          <BaseInput 
            type="text" 
            name="adjustdate" 
            id="adjustdate" 
            v-model="product.AdjustDate"
            disabled>
          </BaseInput>
        </div>
      </div>

      
      <div class="text-center">
        <button type="button" class="btn btn-info btn-fill float-right" @click.prevent="updateProduct">
          Cập nhật sản phẩm
        </button>
      </div>
      <div class="clearfix"></div>
    </form>
  </card>
</template>

<script>
import Card from 'src/components/Cards/Card.vue'
import axios from 'axios';
import BaseInput from '../../components/Inputs/BaseInput.vue';

export default {
  components: {
    Card,
    BaseInput
  },
  data() {
    return {
      product: {
        ProductID: '',
        Name: '',
        Code: '',
        UnitCurrency: '',
        UnitPrice: 0,
        CreatedDate: '',
        AdjustDate: '',
        UserIDCreate: '',
        UserIDAdjust: ''
      },
      department: localStorage.getItem('department') || 'Chưa xác định',
      createdByName: '',
      adjustedByName: '',
      unitCurrencies: ['Cái', 'Quyển', 'Cây','Bộ', 'Hộp', 'Thùng', 'Bịch', 'Gói', 'Chiếc'],
      type: ['success', 'danger', 'warning'],
      notifications: {
        topCenter: false
      },
      token: localStorage.getItem('authToken'),
      userID: localStorage.getItem('userId'), // Assuming userID is stored in localStorage
    }
  },
  methods: {
  async navigateBackToTableList() {
    this.$router.push('/admin/table-list');
  },
  async updateProduct() {
    if (this.product.Name && this.product.Code && this.product.UnitCurrency && this.product.UnitPrice) {
      try {
        const now = new Date().toISOString();
          // Updating an existing product
          const payload = {
        productID: this.product.ProductID, // Correct casing
        name: this.product.Name,
        code: this.product.Code,
        unitCurrency: this.product.UnitCurrency,
        unitPrice: this.product.UnitPrice,
        userIDAdjust: this.userID, // Correct casing
      };
        
    const response = await axios.put(`https://localhost:7162/Product`, payload, {
      headers: { Authorization: `Bearer ${this.token}` },
    });

    console.log('Product updated successfully:', response.data);
        const requestsResponse = await axios.get(`https://localhost:7162/Request/requests-by-product/${this.product.ProductID}`, {
          headers: { Authorization: `Bearer ${this.token}` }
        });
        const requests = requestsResponse.data;
        //print requestID list
        console.log("Request IDs:", requests.map(request => request.requestID));
        const request_ID_list = requests.map(request => request.requestID);
        for (const request_ID of request_ID_list) {
  try {
    const payload = { requestID: request_ID };
    console.log('Sending payload:', payload);

    await axios.post('https://localhost:7162/Request/recalculate-total-price', payload, {
      headers: { Authorization: `Bearer ${this.token}` },
    });
  } catch (error) {
    console.error(
  "Error recalculating total price for request ID ".concat(request_ID, ":"))
  }
}
        //console.log("Response:", response.data);
        const u_id = localStorage.getItem('userId')
        // Fetch the username by userID
        const usernameResponse = await axios.get(`https://localhost:7162/User/getNameById${u_id}`, {
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
          message: `Sản phẩm ${this.product.Name} đã được chỉnh sửa bởi ${username}.`,
          requestID: this.product.ProductID,
          sender: this.userID,
        }));

        for (const notification of notifications) {
          //console.log(`Sending notification: ${JSON.stringify(notification)}`);
          await axios.post('https://localhost:7162/Notification', notification, {
            headers: { Authorization: `Bearer ${this.token}` },
          });
        }

        await this.notifySuccess('bottom', 'right');
        this.navigateBackToTableList();
      } catch (error) {
        console.error("Error updating product:", error);
        this.notifyError('bottom', 'right');
      }
    } else {
      this.notifyWarning('bottom', 'right');
    }
  },
  async notifySuccess(verticalAlign, horizontalAlign) {
    const color = 0
    this.$notifications.notify(
      {
        message: `<span>Cập nhật sản phẩm thành công</span>`,
        icon: 'nc-icon nc-app',
        horizontalAlign: horizontalAlign,
        verticalAlign: verticalAlign,
        type: this.type[color]
      })
  },
  async notifyError(verticalAlign, horizontalAlign) {
    const color = 1
    this.$notifications.notify(
      {
        message: `<span>Cập nhật sản phẩm thất bại, có lỗi xảy ra</span>`,
        icon: 'nc-icon nc-app',
        horizontalAlign: horizontalAlign,
        verticalAlign: verticalAlign,
        type: this.type[color]
      })
  },
  async notifyWarning(verticalAlign, horizontalAlign) {
    const color = 2
    this.$notifications.notify(
      {
        message: `<span>Vui lòng nhập đủ thông tin</span>`,
        icon: 'nc-icon nc-app',
        horizontalAlign: horizontalAlign,
        verticalAlign: verticalAlign,
        type: this.type[color]
      })
  },
  async fetchUserName(userID, type) {
    try {
      const response = await axios.get(`https://localhost:7162/User/getNameById${userID}`, {
        headers: { Authorization: `Bearer ${this.token}` }
      });
      if (type === 'created') {
        this.createdByName = response.data;
      } else if (type === 'adjusted') {
        this.adjustedByName = response.data;
      }
    } catch (error) {
      console.error("Error fetching user name:", error);
    }
  },
  
},
mounted() {
    const token = localStorage.getItem('authToken');
    const decodeToken = JSON.parse(atob(token.split('.')[1]));
    this.userID = decodeToken.UserID;
    this.token = token;
    this.userRole = decodeToken.Role;
    this.userName = decodeToken.Name;

    // Retrieve department from query parameters or localStorage
    const departmentFromQuery = this.$route.query.department;
    this.department = departmentFromQuery || localStorage.getItem('department') || 'Chưa xác định';

    // Store the department in localStorage for consistency
    if (departmentFromQuery) {
        localStorage.setItem('department', departmentFromQuery);
    }

    const id = this.$route.params.id;
    if (id) {
        axios
            .get(`https://localhost:7162/Product/${id}`, { headers: { Authorization: `Bearer ${this.token}` } })
            .then((response) => {
                const updatingProduct = response.data;
                this.product.ProductID = updatingProduct.productID;
                this.product.Name = updatingProduct.name;
                this.product.Code = updatingProduct.code;
                this.product.UnitCurrency = updatingProduct.unitCurrency;
                this.product.UnitPrice = updatingProduct.unitPrice;
                this.product.CreatedDate = new Date(updatingProduct.createDate).toLocaleString();
                this.product.AdjustDate = new Date(updatingProduct.adjustDate).toLocaleString();
                this.product.UserIDCreate = updatingProduct.userIDCreate;
                this.product.UserIDAdjust = updatingProduct.userIDAdjust;
                this.fetchUserName(this.product.UserIDCreate, 'created');
                this.fetchUserName(this.product.UserIDAdjust, 'adjusted');
            })
            .catch((error) => {
                console.error('Error fetching product:', error);
            });
    } else {
        console.error('Product ID is undefined!');
    }
}
};
</script>

<style scoped>

</style>