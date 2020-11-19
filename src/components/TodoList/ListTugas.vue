<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>
        <link href="https://fonts.googleapis.com/css?family=Material+Icons" rel="stylesheet">
  
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
                <v-btn color="warning" dark @click="dialogSelesai = true" style="margin: 1vh;">
                    TODO SELESAI
                </v-btn>
                <v-btn color="success" dark @click="dialog = true">
                    Tambah
                </v-btn>
            </v-card-title>
            <v-data-table :headers="headers" :items="todos" :search="search">
                <template v-slot:[`item.actions`]="{ item }">
                    <v-icon small class="mr-2 blue--text" 
                    @click="editItem(item)">
                            edit
                    </v-icon>
                    <v-icon small class="red--text" 
                    @click="deleteItem(item)">
                        delete
                    </v-icon>
                    <input type="checkbox" 
                        v-model="item.delete" 
                        style="margin-left: 50px;" @change="checkbox(item)">
                </template>
                <template v-slot:[`item.priority`]="{ item }">
                    <v-chip v-if="item.priority === 'Penting'" color="red" outlined>{{item.priority}}</v-chip>
                    <v-chip v-if="item.priority === 'Tidak penting'" color="green" outlined>{{item.priority}}</v-chip>
                    <v-chip v-if="item.priority === 'Biasa'" color="blue" outlined>{{item.priority}}</v-chip>
                </template>
            </v-data-table>
        </v-card>

        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline" v-if="tambah == true" >Form Todo</span>
                        <span class="headline" v-else>Edit Form Todo</span>
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
                            :items="['Penting', 'Biasa', 'Tidak penting']"
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
                    <v-btn color="blue darken-1" text @click="cancel"> Cancel</v-btn>
                    <v-btn v-if="tambah == true" color="blue darken-1" text @click="save">Save</v-btn>
                    <v-btn v-else color="blue darken-1" text @click="edit(formTodo)">Save</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        
        <v-card>
            <v-card-title>
                <h4>Delete Multiple</h4>
            </v-card-title>
            <v-card-text align="left">
                <ul v-for="(todos, i) in selected" :key="i">
                    <li>
                       {{todos.task}} 
                    </li>
                </ul>
            </v-card-text>
            <v-card-actions>
                <v-btn color="red" dark @click="hapusSemua">
                Hapus Semua 
            </v-btn>
            </v-card-actions>
        </v-card>
        <v-dialog v-model="dialogDelete" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Yakin ingin menghapus?</span>
                </v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="green darken-1" text @click="cancel">Tidak</v-btn>
                    <v-btn color="red darken-1" text @click="confirmdelete">Ya</v-btn>
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
			dialogDelete: false,
            dialogFinished: false,
            tambah:true,
			headers: [
				{
					text: "Task",
					align: "start",
					sortable: true,
					value: "task",
				},
				{ text: "Priority", value: "priority" },
				{ text: "Note", value: "note" },
				{ text: "Actions", value: "actions" },
			],
			headersFinished: [
				{
					text: "Task",
					align: "start",
					sortable: true,
					value: "task",
				},
				{ text: "Priority", value: "priority" },
				{ text: "Note", value: "note" },
			],
			todos: [
				{
					task: "Bernafas",
					priority: "Penting",
					note: "huffttt",
				},
				{
					task: "Nongkrong",
					priority: "Tidak penting",
					note: "bersama tman2",
				},
				{
					task: "Masak",
					priority: "Biasa",
					note: "masak air 500ml",
				},
			],
			finishedTodos: [],
			formTodo: {
				task: null,
				priority: null,
				note: null,
            },
            selected: [],
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
            this.edititem = null;
            this.tambah = true;
            this.dialogDelete = false;
		},
		resetForm() {
			this.formTodo = {
				task: null,
				priority: null,
				note: null,
			};
		},
		editItem(item) {
            this.editIndex = this.todos.indexOf(item);
            this.tambah = false;
			this.formTodo = {
				task: item.task,
				priority: item.priority,
				note: item.note,
            };
            
            this.edititem = item;
            this.dialog = true;  
        },
        edit(formTodo){
            this.edititem.task = formTodo.task;
            this.edititem.priority = formTodo.priority;
            this.edititem.note = formTodo.note;
            this.cancel();
        },
		deleteItem(item) {
			this.dialogDelete = true;
            this.edititem = item;
		},
		confirmdelete() {
			this.todos.splice(this.todos.indexOf(this.edititem), 1);
            this.dialogDelete = false;
        },
        hapusSemua(){
            this.todos = this.todos.filter(del=>!this.selected.includes(del));
            this.selected = [];
        },
        checkbox(item){
            if(this.selected.includes(item)) {
                this.selected.splice(this.selected.indexOf(item), 1);
            } else {
                this.selected.push(item);
            }
        }
	},
};
</script>