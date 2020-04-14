<template>
    <v-container>
        <v-card max-width="90vw" class="my-3 elevation-10">
            <v-toolbar color="blue-grey darken-3" dark flat>
                <v-icon class="mx-2">$userManagement</v-icon>
                <v-toolbar-title>User Management</v-toolbar-title>
                <v-spacer/>
                <v-tooltip top color="success">
                    <template #activator="{ on }">
                        <v-icon class="mx-2" v-on="on">
                            mdi-plus-box
                        </v-icon>
                    </template>
                    <span>Add new user</span>
                </v-tooltip>
                <v-tooltip top color="success">
                    <template #activator="{ on }">
                        <v-icon class="mx-2" @click="search.visible = !search.visible" v-on="on">
                            mdi-magnify
                        </v-icon>
                    </template>
                    <span>Search the user database</span>
                </v-tooltip>
            </v-toolbar>
            <v-card-text>
                <v-text-field
                        v-model="search.text"
                        v-if="search.visible"
                        label="Search the user database..."
                        flat
                        solo-inverted
                        hide-details
                        clearable
                        clear-icon="mdi-close-circle-outline"
                        class="mb-3"
                ></v-text-field>
                <v-data-table :headers="headers"
                              :items="accounts"
                              :items-per-page="10"
                              :search="search.text"
                >
                    <template v-slot:item.providerData="{ item }">
                        <v-icon v-for="(icon, index) in providerIcons(item)" :key="index" class="mx-1">{{icon}}</v-icon>
                    </template>
                    <template v-slot:item.metadata.creationTime="{ item }">
                        <span>{{item.metadata.creationTime | formattedDate}}</span>
                    </template>
                    <template v-slot:item.metadata.lastSignInTime="{ item }">
                        <span>{{item.metadata.lastSignInTime | formattedDate}}</span>
                    </template>
                </v-data-table>
            </v-card-text>
        </v-card>

    </v-container>
</template>

<script>
    import firebase from '@/firebase';

    export default {
        name: 'UserManagement',
        data() {
            return {
                accounts: [],
                search: {
                    visible: false,
                    text: null
                },
                headers: [
                    {
                        text: 'Email',
                        align: 'start',
                        value: 'email'
                    },
                    {
                        text: 'Providers',
                        value: 'providerData',
                        sortable: false
                    },
                    {
                        text: 'Created',
                        value: 'metadata.creationTime',
                        sort: (a, b) => {
                            return Date.parse(a) - Date.parse(b)
                        }
                    },
                    {
                        text: 'Signed in',
                        value: 'metadata.lastSignInTime',
                        sort: (a, b) => {
                            return Date.parse(a) - Date.parse(b)
                        }
                    },
                    {
                        text: 'User UID',
                        value: 'uid'
                    }
                ]
            };
        },
        mounted() {
            this.getAllAccounts();
        },
        methods: {
            getAllAccounts() {
                const getAllAccounts = firebase.functions.httpsCallable('userManagement');
                getAllAccounts({ action: 'getAllAccounts' })
                    .then(result => {
                        this.accounts = result.data.users;
                    });
            },
            providerIcons(user) {
                return user.providerData.map(provider => {
                    switch (provider.providerId) {
                        case 'google.com':
                            return '$google';
                        case 'facebook.com':
                            return '$facebook';
                        default:
                            return '$email';
                    }
                });
            }
        },
        filters: {
            formattedDate: function(dateString) {
                const date = new Date(Date.parse(dateString));
                const day = date.getDate().toString().length === 1 ? '0' + date.getDate() : date.getDate();
                const month = date.getMonth().toString().length === 1 ? '0' + (date.getMonth() + 1) : (date.getMonth() + 1);
                const year = date.getFullYear();
                const hours = date.getHours().toString().length === 1 ? '0' + date.getHours() : date.getHours();
                const minutes = date.getMinutes().toString().length === 1 ? '0' + date.getMinutes() : date.getMinutes();
                const seconds = date.getSeconds().toString().length === 1 ? '0' + date.getSeconds() : date.getSeconds();

                return `${day}/${month}/${year} ${hours}:${minutes}:${seconds} GMT`
            }
        }
    };
</script>

<style scoped>

</style>