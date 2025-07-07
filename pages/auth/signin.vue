<template>
    <div class="d-flex align-center justify-center" style="height: 100vh;">
        <v-card width="500">
            <v-card-text justify="center">
                <h1 class="text-center my-8"> Login Page </h1>
                <v-form>
                    <v-text-field v-model="name" label= "Username" rounded outlined prepend-icon="mdi-account"> </v-text-field>

                    <v-text-field v-model="email" label="Password" rounded outlined type="password"
                        prepend-icon="mdi-lock"></v-text-field>

                    <v-btn color="blue" rounded block>
                        Login 
                    </v-btn>
                    <div class="mx-3 my-3 d-flex align-center justify-center"> OR </div>
                    <v-btn color="green" @click = "signinWithGoogle()" class="text-capatilized" rounded block > <v-icon left> mdi-google ></v-icon> Sign in With Google </v-btn>
                </v-form>
            </v-card-text>
        </v-card>
    </div>
</template> 
<script>
export default {
    layout: "auth",
    middleware: "guest",
    methods: {
        signinWithGoogle(){
            this.$auth.loginWith('google')
        }
    }
}
</script>