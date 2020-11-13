<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List Tugas</h3>
        <div class="d-flex flex-row mb-6">
             <v-card width="500px">
                <v-card-title>
                    <v-text-field 
                    v-model ="search"
                    append-icon ="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details></v-text-field>
                    <v-spacer></v-spacer>
                    <v-btn color="success" dark @click="dialog = true">
                        Tambah
                    </v-btn>
                </v-card-title>
                <v-data-table :headers ="headers" :items="todos" :search="search" >
                    <template v-slot:[`item.priority`]="{item}">

                            <v-chip
                                class="ma-2"
                                color="red"
                                v-if="item.priority=='Penting'"
                                outlined
                                >
                                    {{item.priority}}
                            </v-chip>
                            <v-chip
                                class="ma-2"
                                color="green"
                                outlined
                                v-else-if="item.priority=='Tidak penting'"
                                >
                                    {{item.priority}}
                            </v-chip>
                            <v-chip
                                class="ma-2"
                                color="primary"
                                outlined
                                v-else-if="item.priority=='Biasa'"
                                >
                                    {{item.priority}}
                            </v-chip>
                        
                    </template>
                    <template v-slot:[`item.actions`]="{item}"> 
                        <v-icon  left color="blue"  @click ="showItem(item)">

                             mdi-file-outline

                        </v-icon>
                    </template>
                    <template v-slot:[`item.checkbox`]="{item}" >
                        <v-checkbox v-model="arrayHapusSemua" :value="item">    

                        </v-checkbox>
                    </template>
                </v-data-table>
            </v-card>
            <div v-show="index!=null" class="ml-3" style="background-color:gray; height: 200px ; color:white" >
            <p >{{nama}}</p>
                <v-chip
                    class="ma-2"
                    color="red"
                    v-if="prioritas=='Penting'"
                    >
                    {{prioritas}}
                </v-chip>
                <v-chip
                    class="ma-2"
                    color="green"
                    v-else-if="prioritas=='Tidak penting'"
                    >
                    {{prioritas}}
                </v-chip>
                <v-chip
                    class="ma-2"
                    color="primary"
                    v-if="prioritas=='Biasa'"
                    
                    >
                    {{prioritas}}
                </v-chip>
            <p >{{catatan}}</p>
            <v-card>
                
            </v-card>
            <v-icon left color="blue" @click="edit">

                mdi-pencil

            </v-icon>
            <v-icon left color="red" @click="hapus=true">

                mdi-delete

            </v-icon>   
            </div>
        </div>

        <v-card persistent max-width="600px" v-show="arrayHapusSemua.length">
            <v-btn
            depressed
            color="error"
            @click="deleteAll"
            >
            Hapus Semua
            </v-btn>
            <ul v-for="item in arrayHapusSemua" :key="item">
            <li>{{item.task}}</li>
        </ul>
        </v-card>

        <v-dialog v-model="hapus" persistent max-width="600px">
            <v-card>
                <v-card-title class="headline">
                    Yakin Ingin Menghapus?
                </v-card-title>
                <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                    color="green darken-1"
                    text
                    @click="hapus=false"
                >
                    Tidak
                </v-btn>
                <v-btn
                    color="green darken-1"
                    text
                    @click="deleteItem"
                >
                    Ya
                </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
       


        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form To Do</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                        v-model="formTodo.task"
                        label="Task"
                        required>
                            
                        </v-text-field>
                        <v-select 
                        v-model="formTodo.priority" :items ="['Penting','Biasa','Tidak penting']"
                        label="Priority"
                        required>

                        </v-select>
                        <v-textarea
                        v-model="formTodo.note"
                        label="Note"
                        required> 

                        </v-textarea>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">
                        Cancel
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="save">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>


        <v-dialog v-model="editing" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Edit</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                        v-model="formTodo.task"
                        label="Task"
                    
                        required>
                            
                        </v-text-field>
                        <v-select 
                        v-model="formTodo.priority" :items ="['Penting','Biasa','Tidak penting']"
                        label="Priority"
                        required>

                        </v-select>
                        <v-textarea
                        v-model="formTodo.note"
                        label="Note"
                        required> 
                            
                        </v-textarea>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancelEdit">
                        Cancel
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="editItem">
                        Edit
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        
        
    </v-main>
</template>
<script>
    export default{
        name:"List",
        data(){
            return{
                arrayHapusSemua:[],
                search:null,
                dialog:false,
                editing:false,
                test: null,
                detail:false,
                index:null,
                nama:null,
                prioritas:null,
                catatan:null,
                hapus:false,
                headers:[
                    {
                        text:"Task",
                        align:"start",
                        sortable:true,
                        value:"task",
                    },
                    {text:"Priority", value:"priority"},
                    {text :"Actions", value:"actions"},
                    {text:" ", value:"checkbox"},
                ],
                todos: [
                    {
                        task:"bernafas",
                        priority:"Penting",
                        note:"huftttt",
                    },
                    {
                        task:"nongkrong",
                        priority:"Tidak penting",
                        note:"bersama teman2",
                    },
                    {
                        task:"masak",
                        priority:"Biasa",
                        note:"masak air 500ml",
                    },
                ],
                formTodo:{
                    task:null,
                    priority:null,
                    note:null,
                },
                hapusSemua:[
                    {
                        task:null,
                        priority:null,
                        note:null,
                    }
                ]
            };
        },
        methods:{
            save(){
                this.todos.push(this.formTodo);
                this.resetForm();
                this.dialog=false;
            },
            cancel(){
                this.resetForm();
                this.dialog=false;
            },
            cancelEdit(){
                this.resetForm();
                this.editing=false;
            },
            resetForm(){
                this.formTodo={
                    task:null,
                    priority:null,
                    note:null,
                };
            },
            edit(){
                this.editing=true;
                this.formTodo.task=this.nama;
                this.formTodo.priority=this.prioritas;
                this.formTodo.note=this.catatan;

                
            },
            editItem(){
                
                this.todos.splice(this.index,1);
                this.todos.splice(this.index,0,this.formTodo);
                this.index=null;
                this.nama=this.formTodo.task;
                this.prioritas=this.formTodo.priority;
                this.catatan=this.formTodo.note;
                this.resetForm();
                this.editing=false;
                this.detail=false;
            },
            deleteItem(){
                this.todos.splice(this.index,1);
                this.detail=false;
                this.hapus=false;
                this.index=null;
                this.nama=null;
                this.prioritas=null;
                this.catatan=null;
            },
            deleteAll(){
                this.arrayHapusSemua.forEach(element => {
                    if(this.nama==element.task)
                    {
                        this.index=null;
                    }
                });
                this.arrayHapusSemua.forEach(element => {
                    const tes = this.todos.indexOf(element);
                    this.todos.splice(tes,1);
                   
                    
                });
                this.arrayHapusSemua=[];
            },
            showItem(item){
                this.detail=true;
                const index = this.todos.indexOf(item);
                this.index=index;
                this.nama = item.task;
                this.prioritas=item.priority;
                this.catatan = item.note;
            },
        },
    };
</script>