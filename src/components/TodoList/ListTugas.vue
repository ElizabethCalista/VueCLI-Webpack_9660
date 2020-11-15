<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List TGS</h3>

        <v-card>
            <v-card-title>
                <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details
                ></v-text-field>
                <v-spacer></v-spacer>
                <v-col
                    cols="12"
                    sm="5">
                    <v-select
                        v-model="sort" 
                        :items="['Penting', 'Tidak penting']"
                        label="Urutkan"
                        @change="sortingPriority(sort)"
                        hide-details
                        outlined dense
                    ></v-select>
                </v-col>
                <v-spacer></v-spacer>
                <v-btn color="success" dark @click="dialog = true">
                    Tambah
                </v-btn>
            </v-card-title>
            <v-data-table 
                :headers="headers" 
                :items="todos" 
                :search="search" 
                :single = "search" 
                :expanded.sync = "expanded" 
                item-key = "note" 
                show-expand class = "elevation-1">
                <template v-slot:[`item.priority`]="{ item }">
                    <v-chip v-if="item.priority == 'Penting'"  class="ma-2"  color="red" label outlined>
                        {{ item.priority }}
                    </v-chip>
                    <v-chip v-if="item.priority == 'Biasa'" class="ma-2" color="blue" label outlined>
                        {{ item.priority }}
                    </v-chip>
                    <v-chip v-if="item.priority == 'Tidak penting'" class="ma-2" color="green" label outlined>
                        {{ item.priority }}
                    </v-chip>
                </template>
                <template v-slot:[`item.actions`]="{ item }">
                    <v-icon 
                        class="mr-2" 
                        @click="editItem(item)"
                        color="blue"
                    >
                         mdi-pencil
                    </v-icon>
                    <v-icon  
                        class="mr-2"  
                        @click="deleteItem(item)"
                        color="red"
                    >
                        mdi-delete
                    </v-icon>
                </template>
                <template v-slot:expanded-item="{ headers, item }">
                    <td :colspan="headers.length">
                        <br>
                        <h2 class="text-left"> Note : </h2>
                        <p class="text-left">{{ item.note }}</p>
                    </td>
                </template>
                <template v-slot:[`item.checks`]="{ item }">
                    <v-checkbox multiple :key="item" @click.capture.stop="choose(item)"/>              
                </template>
            </v-data-table>
        </v-card>

        <v-card v-if="tempDelete.length" class="pa-5 mt-5">
            <p>Delete Multiple : </p>
            <ul v-for="del in tempDelete" :key="del">
                <li>{{ del.task }}</li>
            </ul>
            <br>
            <v-btn color="red" dark class="mr-2" @click="deletedAll">HAPUS SEMUA</v-btn>
        </v-card>

    <v-dialog v-model="dialog" persistent max-width="600px">
        <v-card>

            <v-card-title>
                <span class="headline">Form Todo</span>
            </v-card-title>
            <v-card-text>
                <v-container>
                    <v-text-field
                        v-model="formTodo.task"
                        label="Task"
                        required
                    ></v-text-field>

                    <v-select
                        v-model="formTodo.priority"
                        :items="[
                            'Penting', 'Biasa', 'Tidak penting'
                        ]"
                        label="Priority"
                        required
                    ></v-select>

                    <v-textarea
                        v-model="formTodo.note"
                        label="Note"
                        required
                    ></v-textarea>
                </v-container>
            </v-card-text>

            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="cancel">
                    Cancel
                </v-btn>
                <v-btn color="blue darken-1" text @click="updateItem">
                    Save
                </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>

      <v-dialog v-model="dialogdelete" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Anda yakin ingin menghapus?</span>
                </v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="green" text @click="deleteNo">
                        NO
                    </v-btn>
                    <v-btn color="red" text @click="deleteYes">
                        YES
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
</v-main>
</template>
<script>
export default {
    name: "List",
    data() {
        return {
            search: null,
            dialog: false,
            dialogdelete: false,
            temp: null,
            filter: null,
            expanded: [],
            tempDelete: [],
            headers: [
                {
                    text: "Task",
                    align: "start",
                    sortable: true,
                    value: "task",
                },
                
                { text: "Priority", value: "priority" },
                { text: "Actions", value:"actions" },
                { text: "", value: "checks" },
            ],
            todos: [
                {
                    task: "bernafas",
                    priority: "Penting",
                    note: "huffttt",
                },
                {
                    task: "nongkrong",
                    priority: "Tidak penting",
                    note: "bersama tman2",
                },
                {
                    task: "masak",
                    priority: "Biasa",
                    note: "masak air 500ml",
                },
            ],
            formTodo: {
                task: null,
                priority: null,
                note: null,
            },
        };
    },
    methods: {
        save() {
            this.todos.push(this.formTodo);
            this.resetForm();
            this.dialog = false;
        },
        cancel() {
            this.resetForm();
            this.dialog = false;
        },
        resetForm() {
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
            };
        },
        editItem(item){
            this.formTodo = {
                task: item.task,
                priority: item.priority,
                note:  item.note,
            }
            this.temp = item;
            this.dialog = true;
        },
        updateItem(){
            this.save();
            this.todos = this.todos.filter(todo => todo !== this.temp);
            this.temp = null;
            this.dialog = false;
        },
        deleteItem(item) {
            this.dialogdelete = true;
            this.temp = item;
        },
        deleteNo(){
            this.temp = null;
            this.dialogdelete = false;
        },
        deleteYes(){
            this.todos = this.todos.filter(todo => todo !== this.temp);
            this.temp = null;
            this.dialogdelete = false;
        },
        sortingPriority(sort){
            function compare(x,y){
                if(sort == "Penting"){
                    if(x.priority == "Penting")
                        return -1;
                    if(x.priority == "Tidak penting")
                        return 1;
                    if(x.priority == "Biasa" && y.priority == "Penting")
                        return 1;
                    if(x.priority == "Biasa" && y.priority == "Tidak penting")
                        return -1;
                }
                else if(sort == "Tidak penting"){
                    if(x.priority == "Tidak penting")
                        return -1;
                    if(x.priority == "Biasa" && y.priority == "Tidak Penting")
                        return -1;
                    if(x.priority == "Penting")
                        return 1;
                    if(x.priority == "Biasa" && y.priority == "Penting")
                        return -1;
                }
            }
            return this.todos.sort(compare);
        },
        choose(item) {
            this.tempDelete.includes(item) ? 
            this.tempDelete.splice(this.tempDelete.indexOf(item), 1) :
            this.tempDelete.push(item);                
        },
        deletedAll() {
            for (var i=0; i < this.tempDelete.length; i++) {
                if (this.formTodo.task == this.tempDelete[i].task) {
                    this.formTodo = {
                        task: null,
                        priority: null,
                        note: null,
                    };
                }
                this.todos.splice(this.todos.indexOf(this.tempDelete[i]), 1);
            }
            this.tempDelete = [];
        },
    },
};
</script>