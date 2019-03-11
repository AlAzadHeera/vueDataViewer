<template>
    <div class="customer">
        <div class="row justify-content-center">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <div class="row">
                            <div class="col-md-6">
                                <h4>Customers</h4>
                            </div>
                            <div class="col-md-6">
                                <button class="btn btn-primary ml-4" style="float: right" type="button" @click="reloadData()">Refresh <i class="fas fa-sync"></i></button>
                                <button class="btn btn-primary" style="float: right" type="button" @click="create()">Add <i class="fas fa-plus"></i></button>
                            </div>
                        </div>
                    </div>

                    <div class="card-body">

                        <div class="mb-3">
                            <div class="row">
                                <div class="col-md-2">
                                    <strong>Search By</strong>
                                </div>
                                <div class="col-md-3">
                                    <select v-model="queryField" class="form-control" name="" id="fields">
                                        <option value="name">Name</option>
                                        <option value="email">Email</option>
                                        <option value="phone">Phone</option>
                                        <option value="address">Address</option>
                                        <option value="total">Total</option>
                                    </select>
                                </div>
                                <div class="col-md-7">
                                    <input v-model="query" type="text" class="form-control" placeholder="Search">
                                </div>
                            </div>
                        </div>

                        <div class="table-responsive">
                            <table class="table table-hover table-bordered table-dark table-striped">
                                <thead>
                                <tr>
                                    <th scope="col">#</th>
                                    <th scope="col">Name</th>
                                    <th scope="col">Email</th>
                                    <th scope="col">Phone</th>
                                    <th scope="col">Totla</th>
                                    <th scope="col" class="text-center">Action</th>
                                </tr>
                                </thead>
                                <tbody>
                                <tr v-show="customers.length" v-for="(customer, index) in customers" :key="customer.id">
                                    <th scope="row"> {{ index + 1 }} </th>
                                    <td> {{ customer.name}} </td>
                                    <td>{{ customer.email }}</td>
                                    <td>{{ customer.phone}}</td>
                                    <td>{{ customer.total}}</td>
                                    <td class="text-center">
                                        <button type="button" class="btn btn-sm btn-success"><i class="fas fa-eye"></i></button>
                                        <button type="button" class="btn btn-sm btn-info" @click="edit(customer)"><i class="fas fa-edit"></i></button>
                                        <button type="button" class="btn btn-sm btn-danger" @click="destroy(customer)"><i class="fas fa-trash-alt"></i></button>
                                    </td>
                                </tr>
                                <tr v-show="!customers.length">
                                    <td colspan="6">
                                    <div class="alert alert-danger">
                                        Sorry :( No Data Found!!
                                    </div>
                                    </td>
                                </tr>
                                </tbody>
                            </table>
                            <pagination v-if="pagination.last_page > 1" :pagination="pagination" :offset="5" @paginate="query === '' ? getData() : searchData"></pagination>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="customerModalLong" tabindex="-1" role="dialog" aria-labelledby="customerModalLongTitle" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="customerModalLongTitle">{{ editMode ? "Edit" : "Add New" }} Customer</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? update() : store()" @keydown="form.onKeydown($event)">
                    <div class="modal-body">
                        <alert-error :form="form"></alert-error>
                        <div class="form-group">
                            <label>Name</label>
                            <input v-model="form.name" type="text" name="name"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                            <has-error :form="form" field="name"></has-error>
                        </div>
                        <div class="form-group">
                            <label>Email</label>
                            <input v-model="form.email" type="email" name="email"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                            <has-error :form="form" field="email"></has-error>
                        </div>
                        <div class="form-group">
                            <label>Phone</label>
                            <input v-model="form.phone" type="text" name="phone"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('phone') }">
                            <has-error :form="form" field="phone"></has-error>
                        </div>
                        <div class="form-group">
                            <label>Address</label>
                            <textarea v-model="form.address" name="address"
                                      class="form-control" :class="{ 'is-invalid': form.errors.has('address') }"></textarea>
                            <has-error :form="form" field="address"></has-error>
                        </div>
                        <div class="form-group">
                            <label>Total</label>
                            <input v-model="form.total" type="text" name="total"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('total') }">
                            <has-error :form="form" field="total"></has-error>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button :disabled="form.busy" type="submit" class="btn btn-primary">Save changes</button>
                    </div>
                    </form>
                </div>
            </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="detailsModalLong" tabindex="-1" role="dialog" aria-labelledby="detailsModalLongTitle" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="detailsModalLong">Customer Details</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? update() : store()" @keydown="form.onKeydown($event)">
                        <div class="modal-body">
                            <alert-error :form="form"></alert-error>
                            <div class="form-group">
                                <label>Name</label>
                                <input v-model="form.name" type="text" name="name"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Email</label>
                                <input v-model="form.email" type="email" name="email"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                                <has-error :form="form" field="email"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Phone</label>
                                <input v-model="form.phone" type="text" name="phone"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('phone') }">
                                <has-error :form="form" field="phone"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Address</label>
                                <textarea v-model="form.address" name="address"
                                          class="form-control" :class="{ 'is-invalid': form.errors.has('address') }"></textarea>
                                <has-error :form="form" field="address"></has-error>
                            </div>
                            <div class="form-group">
                                <label>Total</label>
                                <input v-model="form.total" type="text" name="total"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('total') }">
                                <has-error :form="form" field="total"></has-error>
                            </div>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                            <button :disabled="form.busy" type="submit" class="btn btn-primary">Save changes</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
        <vue-progress-bar></vue-progress-bar>
        <vue-snotify></vue-snotify>
    </div>
</template>

<script>
    export default {
        data(){
            return {
                editMode: false,
                query:'',
                queryField: 'name',
                customers: [],
                form: new Form({
                   id: '',
                   name: '',
                   email: '',
                   phone: '',
                   address: '',
                   total: '',
                }),
                pagination: {
                    current_page: 1,
                }
            }
        },

        watch: {
          query: function (newQ, old) {
              if (newQ === '') {
                  this.getData()
              }else{
                  this.searchData()
              }
          }  
        },

        mounted() {
            console.log('Component mounted.')
            this.getData();
        },

        methods: {
            getData(){
                this.$Progress.start()
                axios.get('/api/customer?page='+this.pagination.current_page)
                    .then(response => {
                        this.customers = response.data.data
                        this.pagination = response.data.meta
                        this.$Progress.finish()
                    })
                    .catch(error => {
                        console.log(error)
                        this.$Progress.fail()
                    })
            },
            searchData(){
                this.$Progress.start()
                axios.get('/api/search/customer/'+this.queryField+'/'+this.query+'?page='+this.pagination.current_page)
                    .then(response => {
                        this.customers = response.data.data
                        this.pagination = response.data.meta
                        this.$Progress.finish()
                    })
                    .catch(error => {
                        console.log(error)
                        this.$Progress.fail()
                    })
            },
            reloadData(){
                this.getData()
                this.query = ''
                this.queryField = 'name'
                this.$snotify.success('Data Refreshed Successfully!!','Success')
            },
            create(){
                this.editMode = false
                this.form.reset()
                this.form.clear()
                $('#customerModalLong').modal('show');
            },
            store(){
                this.$Progress.start()
                this.form.busy = true
                this.form.post('/api/customer')
                    .then(response => {
                        this.getData()
                        $('#customerModalLong').modal('hide');
                        
                        if (this.form.successful){
                            this.$Progress.finish()
                            this.$snotify.success('Customer Saved Successfully','Success')
                        }else{
                            this.$Progress.fail()
                            this.$snotify('Something Went Wrong','Error')
                        }
                        
                    })
                    .catch(error => {
                        this.$Progress.fail()
                        console.log(error)
                    })
            },
            edit(customer){
                this.editMode = true;
                this.form.reset()
                this.form.clear()
                this.form.fill(customer)
                $('#customerModalLong').modal('show');
            },
            update(){
                this.$Progress.start()
                this.form.busy = true
                this.form.put('/api/customer/' + this.form.id)
                    .then(response => {
                        this.getData()
                        $('#customerModalLong').modal('hide');

                        if (this.form.successful){
                            this.$Progress.finish()
                            this.$snotify.success('Customer Updated Successfully','Success')
                        }else{
                            this.$Progress.fail()
                            this.$snotify('Something Went Wrong','Error')
                        }

                    })
                    .catch(error => {
                        this.$Progress.fail()
                        console.log(error)
                    })
            },
            destroy(customer){
                this.$snotify.clear();
                this.$snotify.confirm(
                    "You will not be able to recover this data!",
                    "Are you sure?",
                    {
                        showProgressBar: false,
                        closeOnClick: false,
                        pauseOnHover: true,
                        buttons: [
                            {
                                text: "Yes",
                                action: toast => {
                                    this.$snotify.remove(toast.id);
                                    this.$Progress.start();
                                    axios.delete("/api/customer/" + customer.id)
                                        .then(response => {
                                            this.getData();
                                            this.$Progress.finish();
                                            this.$snotify.success(
                                                "Customer Successfully Deleted",
                                                "Success"
                                            );
                                        })
                                        .catch(e => {
                                            this.$Progress.fail();
                                            console.log(e);
                                        });
                                },
                                bold: true
                            },
                            {
                                text: "No",
                                action: toast => {
                                    this.$snotify.remove(toast.id);
                                },
                                bold: true
                            }
                        ]
                    }
                );
            }
        }
    }
</script>
