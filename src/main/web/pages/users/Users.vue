<template>
    <v-card>
        <v-card-actions>
            <v-btn flat @click.stop="creationDialog = !creationDialog">创建</v-btn>
        </v-card-actions>

        <v-data-table
                hide-actions
                no-data-text="没有数据"
                :headers="headers"
                :items="items">
            <template slot="items" slot-scope="props">
                <td>{{props.item.email}}</td>
                <td>{{props.item.created_at}}</td>
                <td>{{props.item.updated_at}}</td>
                <td align="center">
                    <a @click="resetPwdDialog = true; standbyEmail = props.item.email">
                        <v-icon color="warning">fas fa-redo-alt</v-icon>
                    </a>
                    <a @click="deleteUser(props.item.email)">
                        <v-icon color="red">fas fa-trash-alt</v-icon>
                    </a>
                </td>
            </template>
        </v-data-table>

        <div class="text-xs-right pt-2">
            <v-pagination v-model="pagination.page" :length="pagination.totalPages" @input="inputPage"></v-pagination>
        </div>

        <!-- dialog -->
        <v-dialog v-if="creationDialog" v-model="creationDialog" max-width="800px">
            <d-creation-user @finish="creationDialog = false; loadUsers()"></d-creation-user>
        </v-dialog>

        <v-dialog v-if="resetPwdDialog" v-model="resetPwdDialog" max-width="800px">
            <d-reset-user-pwd :email="standbyEmail" @finish="resetPwdDialog = false"></d-reset-user-pwd>
        </v-dialog>

    </v-card>
</template>
<script>
    import axios from 'axios'
    import DCreationUser from "../../components/users/DCreationUser.vue"
    import DResetUserPwd from "../../components/users/DResetUserPwd.vue"

    export default {
        components: {
            DResetUserPwd,
            DCreationUser},
        data: () => ({
            headers: [
                {
                    text: 'e-mail',
                    value: 'email',
                    sortable: false
                },
                {
                    text: '创建时间',
                    value: 'created_at',
                    sortable: false
                },
                {
                    text: '修改时间',
                    value: 'updated_at',
                    sortable: false
                },
                {
                    text: '操作',
                    value: 'ops',
                    align: 'center',
                    sortable: false,
                    width: 130
                }
            ],
            items: [],
            pagination: {
                rowsPerPage: 15,
                page: 1,
                totalItems: 0,
                totalPages: 0
            },
            creationDialog: false,
            resetPwdDialog: false,
            standbyEmail: ''
        }),
        mounted() {
            this.loadUsers()
        },
        methods: {
            loadUsers() {
                let p = this.pagination
                axios.get(`/api/admins/users?page=${p.page}&size=${p.rowsPerPage}`).then(response => {
                    this.items = response.data.items
                    p.totalItems = response.data.total_items
                    p.totalPages = response.data.total_pages
                })
            },
            inputPage(v) {
                this.pagination.page = v
                this.loadUsers()
            },
            deleteUser(email) {
                this.$confirm(`确认删除用户 <span class="red--text">${email}</span>`, () => {
                    axios.delete(`/api/admins/users/${email}`).then(() => {
                        this.$notice('删除成功', {top: true, color: 'success'})
                        this.loadUsers()
                    })
                })
            }
        }
    }
</script>