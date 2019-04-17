Ở phần trước, tôi đã giới thiệu cũng như cùng quý bạn đọc tìm hiểu về những định nghĩa, cấu trúc, cách thức hoạt động, làm quen với cấu trúc, ..của thư mục js còn tương đối mới mẻ đó là Nest.js. Hôm nay, tôi cùng các bạn sẽ tiến đi vào thực hành việc code một ứng dụng sử dụng Nest.js để các bạn có thể hiểu rõ hơn về thư viện này cũng như là tính hữu ích của nó đối với lập trình viên chúng ta.

Bài toán tôi lựa chọn để code là một bài toán đơn giản, chúng ta chỉ hiển thị danh sách user dưới dạng list, và cho phép thêm sửa xóa user. Chúng ta sẽ xử dụng NestJs để dev backend API, còn component view thì sử dụng VueJs, cơ sở dữ liệu ở đây tôi chọn MongoDB. 

## Cài đặt NestJS và các package liên quan

Trước hết, bạn cần cài CLI trên máy của mình bằng câu lệnh

```
npm install -g @nestjs/cli
```

Sau khi cài đặt thành công, bạn có thể sử dụng dòng lệnh _nest_ để truy cập vào dự án và tạo NestJS project mới. Tạo mới một project bằng câu lệnh sau

```
nest new customer-list-app-backend
```

Sau khi chạy xong, chuyển đường dẫn của bạn vào thư mục của project và cài thêm package mongoDB

```
cd customer-list-app-backend

npm install --save @nestjs/mongoose mongoose
```

## Chạy ứng dụng

Khơi chạy ứng dụng bằng dòng lệnh

```
npm run start
```

Câu lệnh trên sẽ chạy ứng dụng của bạn trên cổng mặc định 3000, bạn có thể kiểm tra bằng cách truy cập vào đường dẫn [http://localhost:3000/](http://localhost:3000/)

## Connecting tới MongoDB

Ở bước trên, chúng ta đã cài đặt mongoDB vào máy tính của mình, để chạy database, hãy bật terminal của bạn lên và chạy dòng lệnh

```
run sudo mongod
```

Sau khi, database đã được khởi chạy, để connect project với database, bạn hãy mở file _src/app.module.ts_và thêm đoạn code sau

```
// ./src/app.module.ts

import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { MongooseModule } from '@nestjs/mongoose';
@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost/customer-app', { useNewUrlParser: true })
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

## Database schema

Tiếp theo, bạn tạo mongodb schema, (hiểu đơn giản nó tương tự như là table trong mysql, giúp bạn lưu trữ dữ liệu vào trong database). Bắt đầu bằng việc tạo folder _customer_ trong thư mục gốc _/src_, sau đó tạo một folder _schemas_ rồi file schema mới với tên _customer.schema.ts_

```
// ./src/customer/schemas/customer.schema.ts
import * as mongoose from 'mongoose';

export const CustomerSchema = new mongoose.Schema({
    first_name: String,
    last_name: String,
    email: String,
    phone: String,
    address: String,
    description: String,
    created_at: { type: Date, default: Date.now }
})
```

Như bạn có thể thấy, dữ liệu được lưu vào database có kiểu là String và Date. 

## Interfaces

Vẫn trong thư mục _customers_ ở trên, tạo một thư mục mới có tên _interfaces_, tạo file _customer.interface.ts_ trong thư mục vưà được tạo với nội dung

```
// ./src/customer/interfaces/customer.interface.ts
import { Document } from 'mongoose';

export interface Customer extends Document {
    readonly first_name: string;
    readonly last_name: string;
    readonly email: string;
    readonly phone: string;
    readonly address: string;
    readonly description: string;
    readonly created_at: Date;
}
```

## Data transfer object (DTO)

DTO sẽ định nghĩa cách dữ liệu được truyền qua network. Để làm việc này tạo thư mục _dto_ trong thưc mục _customers_ thêm file mới _create-customer.dto.ts_ với nội dung sau

```
// ./src/customer/dto/create-customer.dto.ts
export class CreateCustomerDTO {
    readonly first_name: string;
    readonly last_name: string;
    readonly email: string;
    readonly phone: string;
    readonly address: string;
    readonly description: string;
    readonly created_at: Date;
}
```

## Tạo Module

Tạo mới module customer bằng dòng lệnh

```
nest generate module customer
```

Câu lệnh này sẽ tạo ra một file mới với tên _customer.module.ts_  ở trong thư mục _customers_ với code default như sau

```
// ./src/customer/customer.module.ts

import { Module } from '@nestjs/common';
@Module({})
export class CustomerModule {}
```

## Tạo Service

Tạo mới service bằng câu lệnh

```
nest generate service customer
```

Sau khi chạy câu lệnh trên thành công, 2 file mới sẽ được sinh ra

- customer.service.ts: Đây là file service chính với @Injectable()
- customer.service.spec.ts: File này dùng cho unit test

_customer.service.ts_ file này sẽ chưa toàn bộ logic cần xử lý của ứng dụng, trước tiên, hãy mở file này lên và edit với đoạn code

```
// ./src/customer/customer.service.ts
import { Injectable } from '@nestjs/common';
import { Model } from 'mongoose';
import { InjectModel } from '@nestjs/mongoose';
import { Customer } from './interfaces/customer.interface';
import { CreateCustomerDTO } from './dto/create-customer.dto';

@Injectable()
export class CustomerService {
    constructor(@InjectModel('Customer') private readonly customerModel: Model<Customer>) { }
    // fetch all customers
    async getAllCustomer(): Promise<Customer[]> {
        const customers = await this.customerModel.find().exec();
        return customers;
    }
    // Get a single customer
    async getCustomer(customerID): Promise<Customer> {
        const customer = await this.customerModel.findById(customerID).exec();
        return customer;
    }
    // post a single customer
    async addCustomer(createCustomerDTO: CreateCustomerDTO): Promise<Customer> {
        const newCustomer = await this.customerModel(createCustomerDTO);
        return newCustomer.save();
    }
    // Edit customer details
    async updateCustomer(customerID, createCustomerDTO: CreateCustomerDTO): Promise<Customer> {
        const updatedCustomer = await this.customerModel
            .findByIdAndUpdate(customerID, createCustomerDTO, { new: true });
        return updatedCustomer;
    }
    // Delete a customer
    async deleteCustomer(customerID): Promise<any> {
        const deletedCustomer = await this.customerModel.findByIdAndRemove(customerID);
        return deletedCustomer;
    }
}
```

Dừng lại để đọc hiểu một chú nhé. Ở đây, bạn đã import những module cần thiết từ @nestjs/common, mongoose và @nestjs/mongoose. Ngoài ra, bạn cũng đã import đến interfaces và DTO mà bạn đã khai báo ở bước trước đó

Để có thể thực hiện các thao tác, thêm mới, sửa, xóa một cách liền mạch bạn sử dụng @InjectModel để inject Customer modal vào trong _CustomerService_ class

Tiếp theo, là các method sau

- getAllCustomer(): để truy xuất và hiển thị danh sách tất cả customer

- getCustomer(): Lấy thông tin 1 customer dựa vào ID của customver.

- addCustomer(): Thêm mới một customer vào database

- updateCustomer(): Method này đòi hỏi bạn cần truyền vào customver id và data của customer, method này sẽ tiến hành update thông tin của customer trong database bằng thông tin trong data bạn truyền vào

- deleteCustomer(): Method này sẽ giúp bạn xóa một customer khỏi database

## Tạo mới Controller

Tạo mới một controller bằng lệnh sau

```
nest generate controller customer
```

Tương tự như khi tạo service lệnh này cũng tạo ra 2 file mới là customer.controller.ts và customer.controller.spec.ts ở trong thư mục _customers_. Mở file customer.controller.ts và thêm nội dung sau:

```
// ./src/customer/customer.controller.ts
import { Controller, Get, Res, HttpStatus, Post, Body, Put, Query, NotFoundException, Delete, Param } from '@nestjs/common';
import { CustomerService } from './customer.service';
import { CreateCustomerDTO } from './dto/create-customer.dto';

@Controller('customer')
export class CustomerController {
    constructor(private customerService: CustomerService) { }

    // add a customer
    @Post('/create')
    async addCustomer(@Res() res, @Body() createCustomerDTO: CreateCustomerDTO) {
        const customer = await this.customerService.addCustomer(createCustomerDTO);
        return res.status(HttpStatus.OK).json({
            message: "Customer has been created successfully",
            customer
        })
    }

    // Retrieve customers list
    @Get('customers')
    async getAllCustomer(@Res() res) {
        const customers = await this.customerService.getAllCustomer();
        return res.status(HttpStatus.OK).json(customers);
    }

    // Fetch a particular customer using ID
    @Get('customer/:customerID')
    async getCustomer(@Res() res, @Param('customerID') customerID) {
        const customer = await this.customerService.getCustomer(customerID);
        if (!customer) throw new NotFoundException('Customer does not exist!');
        return res.status(HttpStatus.OK).json(customer);
    }
}
```

Trên dây chúng ta đã định nghĩa route cho các phương thức tạo mới, lấy tất cả customer và lấy một customer theo id, tiếp theo hãy định nghĩa phương thức update và delete

```
// ./src/customer/customer.controller.ts
...
@Controller('customer')
export class CustomerController {
    constructor(private customerService: CustomerService) { }
    ...

    // Update a customer's details
    @Put('/update')
    async updateCustomer(@Res() res, @Query('customerID') customerID, @Body() createCustomerDTO: CreateCustomerDTO) {
        const customer = await this.customerService.updateCustomer(customerID, createCustomerDTO);
        if (!customer) throw new NotFoundException('Customer does not exist!');
        return res.status(HttpStatus.OK).json({
            message: 'Customer has been successfully updated',
            customer
        });
    }

    // Delete a customer
    @Delete('/delete')
    async deleteCustomer(@Res() res, @Query('customerID') customerID) {
        const customer = await this.customerService.deleteCustomer(customerID);
        if (!customer) throw new NotFoundException('Customer does not exist');
        return res.status(HttpStatus.OK).json({
            message: 'Customer has been deleted',
            customer
        })
    }
}
```

## Update the customer module

```
// ./src/customer/customer.module.ts

import { Module } from '@nestjs/common';
import { CustomerController } from './customer.controller';
import { CustomerService } from './customer.service';
import { MongooseModule } from '@nestjs/mongoose';
import { CustomerSchema } from './schemas/customer.schema';
@Module({
  imports: [
    MongooseModule.forFeature([{ name: 'Customer', schema: CustomerSchema }])
  ],
  controllers: [CustomerController],
  providers: [CustomerService]
})
export class CustomerModule { }
```

## Enable CORS

```
// ./src/main.ts

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors(); // add this line
  await app.listen(3000);
}
bootstrap();
```

## Create Vue Component

Tạo file component Create với nội dung sau

```
// ./src/components/customer/Create.vue

<template>
   <div>
        <div class="col-md-12 form-wrapper">
          <h2> Create Customer </h2>
          <form id="create-post-form" @submit.prevent="createCustomer">
               <div class="form-group col-md-12">
                <label for="title"> First Name </label>
                <input type="text" id="first_name" v-model="first_name" name="title" class="form-control" placeholder="Enter firstname">
               </div>
               <div class="form-group col-md-12">
                <label for="title"> Last Name </label>
                <input type="text" id="last_name" v-model="last_name" name="title" class="form-control" placeholder="Enter Last name">
               </div>
             <div class="form-group col-md-12">
                <label for="title"> Email </label>
                <input type="text" id="email" v-model="email" name="title" class="form-control" placeholder="Enter email">
            </div>
            <div class="form-group col-md-12">
                <label for="title"> Phone </label>
                <input type="text" id="phone_number" v-model="phone" name="title" class="form-control" placeholder="Enter Phone number">
            </div>
            <div class="form-group col-md-12">
                <label for="title"> Address </label>
                <input type="text" id="address" v-model="address" name="title" class="form-control" placeholder="Enter Address">
            </div>
              <div class="form-group col-md-12">
                  <label for="description"> Description </label>
                  <input type="text" id="description" v-model="description" name="description" class="form-control" placeholder="Enter Description">
              </div>
              <div class="form-group col-md-4 pull-right">
                  <button class="btn btn-success" type="submit"> Create Customer </button>
              </div>           </form>
        </div>
    </div>
</template>
```

Thêm code script

```
// ./src/components/customer/Create.vue

...

<script>
import axios from "axios";
import { server } from "../../helper";
import router from "../../router";
export default {
  data() {
    return {
      first_name: "",
      last_name: "",
      email: "",
      phone: "",
      address: "",
      description: ""
    };
  },
  methods: {
    createCustomer() {
      let customerData = {
        first_name: this.first_name,
        last_name: this.last_name,
        email: this.email,
        phone: this.phone,
        address: this.address,
        description: this.description
      };
      this.__submitToServer(customerData);
    },
    __submitToServer(data) {
      axios.post(`${server.baseURL}/customer/create`, data).then(data => {
        router.push({ name: "home" });
      });
    }
  }
};
</script>
```

Tạo file edit compoent

```
// ./src/components/customer/Edit.vue

<template>
   <div>
        <h4 class="text-center mt-20">
         <small>
         <button class="btn btn-success" v-on:click="navigate()"> View All Customers </button>
         </small>
        </h4>
        <div class="col-md-12 form-wrapper">
          <h2> Edit Customer </h2>
          <form id="create-post-form" @submit.prevent="editCustomer">
               <div class="form-group col-md-12">
                <label for="title"> First Name </label>
                <input type="text" id="first_name" v-model="customer.first_name" name="title" class="form-control" placeholder="Enter firstname">
               </div>
               <div class="form-group col-md-12">
                <label for="title"> Last Name </label>
                <input type="text" id="last_name" v-model="customer.last_name" name="title" class="form-control" placeholder="Enter Last name">
               </div>
             <div class="form-group col-md-12">
                <label for="title"> Email </label>
                <input type="text" id="email" v-model="customer.email" name="title" class="form-control" placeholder="Enter email">
            </div>
            <div class="form-group col-md-12">
                <label for="title"> Phone </label>
                <input type="text" id="phone_number" v-model="customer.phone" name="title" class="form-control" placeholder="Enter Phone number">
            </div>
            <div class="form-group col-md-12">
                <label for="title"> Address </label>
                <input type="text" id="address" v-model="customer.address" name="title" class="form-control" placeholder="Enter Address">
            </div>
              <div class="form-group col-md-12">
                  <label for="description"> Description </label>
                  <input type="text" id="description" v-model="customer.description" name="description" class="form-control" placeholder="Enter Description">
              </div>
              <div class="form-group col-md-4 pull-right">
                  <button class="btn btn-success" type="submit"> Edit Customer </button>
              </div>           </form>
        </div>
    </div>
</template>
<script>
import { server } from "../../helper";
import axios from "axios";
import router from "../../router";
export default {
  data() {
    return {
      id: 0,
      customer: {}
    };
  },
  created() {
    this.id = this.$route.params.id;
    this.getCustomer();
  },
  methods: {
    editCustomer() {
      let customerData = {
        first_name: this.customer.first_name,
        last_name: this.customer.last_name,
        email: this.customer.email,
        phone: this.customer.phone,
        address: this.customer.address,
        description: this.customer.description
      };
      axios
        .put(
          `${server.baseURL}/customer/update?customerID=${this.id}`,
          customerData
        )
        .then(data => {
          router.push({ name: "home" });
        });
    },
    getCustomer() {
      axios
        .get(`${server.baseURL}/customer/customer/${this.id}`)
        .then(data => (this.customer = data.data));
    },
    navigate() {
      router.go(-1);
    }
  }
};
</script>
```

List tất cả customer

```
// ./src/views/Home.vue
<template>
    <div class="container-fluid">
      <div class="text-center">
        <h1>Nest Customer List App Tutorial</h1>
       <p> Built with Nest.js, Vue.js and MongoDB</p>
       <div v-if="customers.length === 0">
            <h2> No customer found at the moment </h2>
        </div>
      </div>

        <div class="">
            <table class="table table-bordered">
              <thead class="thead-dark">
                <tr>
                  <th scope="col">Firstname</th>
                  <th scope="col">Lastname</th>
                  <th scope="col">Email</th>
                  <th scope="col">Phone</th>
                  <th scope="col">Address</th>
                  <th scope="col">Description</th>
                  <th scope="col">Actions</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="customer in customers" :key="customer._id">
                  <td>{{ customer.first_name }}</td>
                  <td>{{ customer.last_name }}</td>
                  <td>{{ customer.email }}</td>
                  <td>{{ customer.phone }}</td>
                  <td>{{ customer.address }}</td>
                  <td>{{ customer.description }}</td>
                  <td>
                    <div class="d-flex justify-content-between align-items-center">
                                <div class="btn-group" style="margin-bottom: 20px;">
                                  <router-link :to="{name: 'Edit', params: {id: customer._id}}" class="btn btn-sm btn-outline-secondary">Edit Customer </router-link>
                                  <button class="btn btn-sm btn-outline-secondary" v-on:click="deleteCustomer(customer._id)">Delete Customer</button>
                                </div>
                              </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
    </div>
</template>
<script>
import { server } from "../helper";
import axios from "axios";
export default {
  data() {
    return {
      customers: []
    };
  },
  created() {
    this.fetchCustomers();
  },
  methods: {
    fetchCustomers() {
      axios
        .get(`${server.baseURL}/customer/customers`)
        .then(data => (this.customers = data.data));
    },
    deleteCustomer(id) {
      axios
        .delete(`${server.baseURL}/customer/delete?customerID=${id}`)
        .then(data => {
          console.log(data);
          window.location.reload();
        });
    }
  }
};
</script>
```

## Update App Vue

```
// ./src/App.vue

<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/create">Create</router-link>
    </div>
    <router-view/>
  </div>
</template>

<style>
...
.form-wrapper {
  width: 500px;
  margin: 0 auto;
}
</style>
```

## Include Bootstrap

```
<!DOCTYPE html>
<html lang="en">
<head>
  ...
  <!-- Add this line -->
  <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css">
  <title>customer-list-app-frontend</title>
</head>
<body>
   ...
</body>
</html>
```

## Setting up routing

```
// ./src/router.js

import Vue from 'vue'
import Router from 'vue-router'
import HomeComponent from '@/views/Home';
import EditComponent from '@/components/customer/Edit';
import CreateComponent from '@/components/customer/Create';
Vue.use(Router)
export default new Router({
  mode: 'history',
  routes: [
    { path: '/', redirect: { name: 'home' } },
    { path: '/home', name: 'home', component: HomeComponent },
    { path: '/create', name: 'Create', component: CreateComponent },
    { path: '/edit/:id', name: 'Edit', component: EditComponent },
  ]
});
```
Mọi thứ đã hoàn thiện, chạy lại lệnh 

```
npm run start
```

v
