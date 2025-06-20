<template>
    <div class="d-flex align-center justify-center" style="height: 100vh;">
        <v-card width="500">
            
            <v-card-text justify="center">
                <h1> Login Page </h1>
                <v-form>
                    <v-text-field v-model="name" label="Email"></v-text-field>

                    <v-text-field v-model="email" label="Password"></v-text-field>

                    <v-btn color="blue" block>
                        Signin
                    </v-btn>
                </v-form>
            </v-card-text>
        </v-card>
    </div>
</template>
<script>
export default {
    layout: "auth"
}
</script>