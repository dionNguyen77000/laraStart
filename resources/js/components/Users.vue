<template>
    <div class="container">
       <div class="row mt-5">
          <div class="col-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">User Table</h3>

                <div class="card-tools">
                    
                  <div class="input-group input-group-sm" style="width: 150px;">
                    <button class="btn btn-success" @click="newModal()">Add New   <i class="fas fa-user-plus"></i></button>
                  
                    <!-- <input type="text" name="table_search" class="form-control float-right" placeholder="Search">

                    <div class="input-group-append">
                      <button type="submit" class="btn btn-default"><i class="fas fa-search"></i></button>
                    </div> -->
                    
                  </div>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <thead>
                    <tr>
                      <th>ID</th>
                      <th>Name</th>
                      <th>Email</th>
                      <th>Type</th>
                      <th>Registerd at</th>
                      <th>Modify</th>
                    </tr>
                  </thead>
                  <tbody>
                      <tr v-for="user in users" :key="user.id">
                          <td>{{user.id}}</td>
                            <td>{{user.name}}</td>
                            <td>{{user.email}}</td>
                            <td>{{user.type | upText}}</td>
                            <td>{{user.created_at | myDate}}</td>
                      
    
                      <td>
                          <a @click="editModal(user)">
                              <i class="fa fa-edit blue"></i>
                          </a>
                          /
                          <a  @click="deleteUser(user.id)">
                              <i class="fa fa-trash red"></i>
                          </a>

                      </td>
                      </tr>
                  </tbody>
                </table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" v-show="!editmode" id="addNewlLabel">Add New</h5>
                <h5 class="modal-title" v-show="editmode" id="addNewlLabel">Update User</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <form @submit.prevent="editmode ? updateUser() : createUser()">
            <div class="modal-body">
                <div class="form-group">
                    <label>Username</label>
                    <input v-model="form.name" type="text" name="name" placeholder="Name"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                    <has-error :form="form" field="name"></has-error>
                </div>

                <div class="form-group">
                    <label>Email</label>
                    <input v-model="form.email" type="text" name="email" placeholder="Email"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                    <has-error :form="form" field="email"></has-error>
                </div>

                <div class="form-group">
                    <label>Short Bio</label>
                    <textarea v-model="form.bio" type="text" name="bio" placeholder="Short bio for user (Optional)"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"> </textarea>
                    <has-error :form="form" field="bio"></has-error>
                </div>

                <div class="form-group">
                    <select name="type" v-model="form.type" id="type" class="form-control" :class="{'is-invalid' : form.errors.has('type')}">
                        <option value="">Select User Role</option>
                        <option value="admin">Admin</option>
                        <option value="user">Standard user</option>
                        <option value="author">Author</option>
                    </select>
                    <has-error :form="form" field="password"></has-error>
                </div>
                <div class="form-group">
                    <label>password</label>
                    <input v-model="form.password" type="password" name="password" placeholder="Password"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                    <has-error :form="form" field="password"></has-error>
                </div>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                <button v-show="editmode" type="submit" class="btn btn-primary">Update</button>
                <button v-show="!editmode" type="submit" class="btn btn-primary">Create</button>
            </div>

            
            </form>
            </div>
        </div>
        </div>
    </div>
    
</template>

<script>
    export default {
        data() {
            return {
            editmode : false,
                users:{},
                form: new Form({
                    id:'',
                    name : '',
                    email : '',
                    password : '',
                    type : '',
                    bio : '',
                    photo : '',
                })
            }
        },
        methods: {
            updateUser(){
                 this.$Progress.start();
                // console.log('edit mode');
                this.form.put('api/user/'+this.form.id)
                .then(() => {
                    //success
                    $('#addNew').modal('hide');
                     swal.fire(
                        'Updated!',
                        'Information has been updated.',
                        'success'
                        )
                    this.$Progress.finish();
                    Fire.$emit('AfterCreated');
                   
                })
                .catch(() => { this.$Progress.fail(); });
            },
            editModal(user){
                this.editmode=true;
                this.form.reset();
                 $('#addNew').modal('show');
                 this.form.fill(user);
            },
            newModal(){
                 this.editmode=false;
                this.form.reset();
                 $('#addNew').modal('show');
            },
            deleteUser(id){
                swal.fire({
                title: 'Are you sure?',
                text: "You won't be able to revert this!",
                type: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                if (result.value) {    
                    // send request to the server
                    this.form.delete('api/user/'+id).then(()=>{
                        swal.fire(
                        'Deleted!',
                        'Your file has been deleted.',
                        'success'
                        )
                        Fire.$emit('AfterCreated');
                    }).catch(()=>{
                        swal.fire('Failed','There were something wrong.',"warning");
                    });
                }
                })
               
            },
            loadUsers(){
                axios.get("api/user").then(({data})=>(this.users = data.data));
            },
            createUser(){
                this.$Progress.start();
                this.form.post('api/user')
                Fire.$emit('AfterCreated');
                $('#addNew').modal('hide');
                toast.fire({
                    type: 'success',
                    title: 'User Created successfully'
                })
                // .then(()=>{
                  
                    this.$Progress.finish();
                // })
                // .catch(() => { this.$Progress.fail(); });
            }
        },
        created() {
            this.loadUsers();
            Fire.$on('AfterCreated',() => {
                this.loadUsers();
            });
            // setInterval(()=>this.loadUsers(), 3000);
        }
    }
</script>
