<template>
    <body>

    <div class="container-md" >

        <div class="main">
            <div class="row">
                <div class="col-md-12 text-left">
                    <h3 style="color:#E95A09">Data Jabatan Pegawai</h3>
                    <!-- <button type="button" class="btn btn-warning ">Save</button> -->
                </div>
                <div class="col-md-12 text-right">
                    <button type="button" class="btn btn-primary" data-toggle="modal" @click="showModal()"  style="background-color:#124EB2">Tambah Jabatan</button>
                    <!-- <button type="button" class="btn btn-warning ">Save</button> -->
                </div>
                </div>
            <div style="margin-top:10px">
            <table class="table align-start border"  >
                <thead class="" >
                <tr>
                    <th>&nbsp;</th>
                    <th>No</th>
                    <th>Jabatan</th>
                    <th>Gaji</th>
                    <th>Tunjangan</th>
                    <th style="text-align: center;">Action</th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="(data,index) in jabatan" :key="data.id">
                        <td>
						    <label class="checkbox-wrap checkbox-success">
								<input type="checkbox">
								<span class="checkmark"></span>
							</label>
                        </td>
                        <td>{{index+1}} </td>
                        <td>{{ data.jabatan }}</td>
                        <td>{{convertToRupiah (data.gaji) }}</td>
                         <td>{{convertToRupiah (data.tunjangan) }}</td>
                        <td style="text-align: center;">
                             <button type="button" class="btn btn-link btn-sm btn-rounded" data-toggle="modal" @click.prevent="showModalEdit(data)">
                                    Edit
                            </button>
                             <button type="button" class="btn btn-link danger btn-sm btn-rounded" style="color:red;" @click.prevent="hapusJabatan(data.id)">
                                    Hapus
                            </button>
                        </td>
                </tr>
            </tbody>
            </table>
            </div>
        </div>

    <div class="modal fade" id="tambahJabatan" tabindex="-1" role="dialog" aria-labelledby="tambahJabatan" aria-hidden="true">
            <div class="modal-dialog modal-lg modal-dialog-centered" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h2 class="modal-title" v-show="!statusmodal"  id="tambahJabatan" style="color:#E95A09">Tambah jabatan Pegawai</h2>
                    <h2 class="modal-title" v-show="statusmodal"  id="tambahJabatan" style="color:#E95A09">Update Jabatan Pegawai</h2>
                    <button type="button" @click="closeModal()" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">×</span>
                    </button>
                </div>
                <form @submit.prevent=" statusmodal ? editJabatan() : addJabatan()" >
                <div class="modal-body" style="padding:30px">
                    <div class="alert alert-danger" style="display:none"></div>
                    
                        <div class="form-group">
                            <label style="font-weight:bold">Jabatan</label>
                            <input v-model="form.jabatan" placeholder="Masukan Jabatan" type="text" class="form-control">
                        </div>
                        <div class="form-group" >
                            <label style="font-weight:bold">Gaji</label>
                            <input v-model="form.gaji" placeholder="Hanya Masukan Angka" type="text" class="form-control">
                        </div>
                         <div class="form-group">
                            <label style="font-weight:bold">Tunjangan</label>
                            <input v-model="form.tunjangan" placeholder="Hanya Masukan Angka" type="text" class="form-control">
                        </div>
                    

                        <div class="modal-footer end">
                        <button type="button" class="btn" style=" border-radius:6px; background-color:#E95A09;color:white;   width:100px " @click="closeModal()" data-dismiss="modal">Batal</button>
                        <button type="submit" class="btn" style=" border-radius:6px; background-color:#124EB2;color:white;  width:100px "  v-show="!statusmodal">Tambah</button>
                         <button type="submit" class="btn" style=" border-radius:6px; background-color:#124EB2;color:white;  width:100px " v-show="statusmodal">Ubah</button>
                     </div>
                       
                    
                </div>
                </form>
            </div>
        </div>
        
    </div>

    </div>
    </body>

</template>


<script>

export default {
    name: "Jabatan",
    data() {
        return {
            jabatan:[],
            statusmodal: false,
            form : new Form({
                id : "",
                jabatan : "",
                gaji : "",
                tunjangan : "",
            }),
            token: localStorage.getItem("token"),
            role: localStorage.getItem('role'),
        }
    },
    created() {
         this.$axios.get('/sanctum/csrf-cookie').then(response => {
            this.$axios.get('/api/alljabatan',{
                headers: {Authorization: "Bearer " + this.token},
            })
                .then(response => {
                    this.jabatan = response.data.data;
                })
                .catch(function (error) {
                    console.error(error);
                });
        })
    },
    methods:{
        showModal() {
            this.statusmodal = false;
            this.form.reset();
            $("#tambahJabatan").modal("show");
        },
        showModalEdit(data) {
            this.statusmodal = true;
            this.form.reset();
            $("#tambahJabatan").modal("show");
            this.form.fill(data);
        },
        closeModal() {
            this.form.reset();
            $("#tambahJabatan").modal("hide");
        },

        addJabatan() {
            this.$axios.get('/sanctum/csrf-cookie').then(response => {
            this.form.post('/api/tambahjabatan', {
                headers : { Authorization: "Bearer " + this.token },
            }).then((response) => {
                if (response.data.success){
                    Swal.fire({
                        icon: "success",
                        title: "Berhasil",
                        text: "Berhasil Menambahkan Jabatan",
                        showConfirmButton: false,
                        timer: 1600,
                    })
                    this.form.reset();
                    $('#tambahJabatan').modal('hide')
                    this.allJabatan()
                }
                    
                    }
                )
                    })
        },
        editJabatan(){
        
              axios.post('/api/updatejabatan',
                {
                    id: this.form.id,
                    jabatan: this.form.jabatan,
                    gaji: this.form.gaji,
                    tunjangan: this.form.tunjangan,
                },
                {
                    headers: { Authorization: "Bearer " + this.token }
                }).then((response) => {
                if (response.data.success){
                    Swal.fire({
                        icon: "success",
                        title: "Berhasil",
                        text: "Berhasil Update Akun Pegawai",
                        showConfirmButton: false,
                        timer: 1600,
                    })
                    this.form.reset();
                    $("#tambahJabatan").modal("hide");
                    this.allJabatan()
                }
            

           })
        },

        hapusJabatan(id){
            this.$axios.get('/sanctum/csrf-cookie').then(response => {
            Swal.fire({
            title: 'Hapus Jabatan ?',
            text: "Anda tidak akan bisa mengembalikannya lagi!",
            icon: 'warning',
            showCancelButton: true,
            confirmButtonColor: '#124EB2',
            cancelButtonColor: '#E95A09',
            confirmButtonText: 'Hapus',
            cancelButtonText: 'Batal',
            }).then((result) => {
                 if (result.isConfirmed) {
                 axios.delete('/api/hapusjabatan/'+id,  {
                headers: { Authorization: "Bearer " + this.token },
                },
                )
                  Swal.fire({
                        icon: "success",
                        title: "Berhasil",
                        text: "Berhasil Menghapus Jabatan Pegawai",
                        showConfirmButton: false,
                        timer: 1600,
                    })
                    };
           
                    this.allJabatan()

            },

            )

            })
        },

        allJabatan(){
             this.$axios.get('/sanctum/csrf-cookie').then(response => {
            this.$axios.get('/api/alljabatan',{
                headers: {Authorization: "Bearer " + this.token},
            })
                .then(response => {
                    this.jabatan = response.data.data;
                })
                .catch(function (error) {
                    console.error(error);
                });
        })
        },

        convertToRupiah(value) {
            // value = value.toString()
            value = parseFloat(value)
            let rupiah = ''
            let valueRev = value.toString().split('').reverse().join('')
            for (let i = 0; i < valueRev.length; i++) {
                if (i % 3 === 0) rupiah += `${valueRev.substr(i, 3)}.`
            }

            const rp = rupiah
                .split('', rupiah.length - 1)
                .reverse()
                .join('')
            // console.log('rupiah', isNaN(rp), rupiah.length, rp, rupiah)
            if (rupiah === 'NaN.' || rupiah === 'NaN') {
                return '...'
            }
            return `Rp ${rp}`
        },

    
        
    },

    mounted() {
    },
    // beforeRouteEnter(to, from, next) {
    //     if (JSON.parse(window.localStorage.getItem("loggedIn"))) {
    //         if(window.localStorage.getItem("role") == 'Manager'){
    //             return next();
    //         } else if (window.localStorage.getItem("role") == 'Admin') {
    //             return next();
    //         } else if (window.localStorage.getItem("role") == 'Pegawai') {
    //             return next();
    //         }
    //     }
    //     next();
    // },


}


</script>


