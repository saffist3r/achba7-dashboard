<template>
  <v-container fluid>
    <validation-observer ref="observer" v-slot="{ invalid }">
      <form @submit.prevent="onSubmit" @reset.prevent="onReset">
        <v-card class="ma-3 pa-3">
          <v-card-title primary-title>
            <div class="headline primary--text">Create User</div>
          </v-card-title>
          <v-card-text>
            <validation-provider v-slot="{ errors }" name="First Name" rules="required">
              <v-text-field
                v-model="firstName"
                label="First Name"
                required
                :error-messages="errors"
              ></v-text-field>
            </validation-provider>
            <validation-provider v-slot="{ errors }" name="Last Name" rules="required">
              <v-text-field
                v-model="lastName"
                label="Last Name"
                required
                :error-messages="errors"
              ></v-text-field>
            </validation-provider>
            <validation-provider
              v-slot="{ errors }"
              rules="required|email"
              name="E-mail"
            >
              <v-text-field
                v-model="email"
                label="E-mail"
                type="email"
                :error-messages="errors"
                required
              ></v-text-field>
            </validation-provider>
            <div class="subheading secondary--text text--lighten-2">
              User is superuser
              <span v-if="isSuperuser">(currently is a superuser)</span
              ><span v-else>(currently is not a superuser)</span>
            </div>
            <v-checkbox v-model="isSuperuser" label="Is Superuser"></v-checkbox>
            <div class="subheading secondary--text text--lighten-2">
              User is active <span v-if="isActive">(currently active)</span
              ><span v-else>(currently not active)</span>
            </div>
            <v-checkbox v-model="isActive" label="Is Active"></v-checkbox>
            <v-row align="center">
              <v-col>
                <validation-provider
                  v-slot="{ errors }"
                  :debounce="100"
                  name="Password"
                  vid="password1"
                  rules="required"
                >
                  <v-text-field
                    v-model="password1"
                    type="password"
                    label="Set Password"
                    :error-messages="errors"
                  ></v-text-field>
                </validation-provider>
                <validation-provider
                  v-slot="{ errors }"
                  :debounce="100"
                  name="Password confirmation"
                  vid="password2"
                  rules="required|confirmed:password1"
                >
                  <v-text-field
                    v-model="password2"
                    type="password"
                    label="Confirm Password"
                    :error-messages="errors"
                  ></v-text-field>
                </validation-provider>
              </v-col>
            </v-row>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn @click="cancel">Cancel</v-btn>
            <v-btn type="reset">Reset</v-btn>
            <v-btn :disabled="invalid" type="submit"> Save </v-btn>
          </v-card-actions>
        </v-card>
      </form>
    </validation-observer>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { IUserProfileCreate } from "@/interfaces";
import { dispatchGetUsers, dispatchCreateUser } from "@/store/admin/actions";
import { ValidationProvider, ValidationObserver, extend } from "vee-validate";
import { required, confirmed, email } from "vee-validate/dist/rules";

// register validation rules
extend("required", { ...required, message: "{_field_} can not be empty" });
extend("confirmed", { ...confirmed, message: "Passwords do not match" });
extend("email", { ...email, message: "Invalid email address" });

@Component({
  components: {
    ValidationObserver,
    ValidationProvider,
  },
})
export default class CreateUser extends Vue {
  $refs!: {
    observer: InstanceType<typeof ValidationObserver>;
  };

  public valid = false;
  public firstName = "";
  public lastName = "";
  public email = "";
  public isActive = true;
  public isSuperuser = false;
  public setPassword = false;
  public password1 = "";
  public password2 = "";

  public async mounted() {
    await dispatchGetUsers(this.$store);
    this.onReset();
  }

  public onReset() {
    this.password1 = "";
    this.password2 = "";
    this.firstName = "";
    this.lastName = "";
    this.email = "";
    this.isActive = true;
    this.isSuperuser = false;
    this.$refs.observer.reset();
  }

  public cancel() {
    this.$router.back();
  }

  public async onSubmit() {
    const success = await this.$refs.observer.validate();
    if (!success) {
      return;
    }

    const updatedProfile: IUserProfileCreate = {
      email: this.email,
    };
    if (this.firstName) {
      updatedProfile.first_name = this.firstName;
    }
    if (this.lastName) {
      updatedProfile.last_name = this.lastName;
    }
    if (this.email) {
      updatedProfile.email = this.email;
    }
    updatedProfile.is_active = this.isActive;
    updatedProfile.is_superuser = this.isSuperuser;
    updatedProfile.password = this.password1;
    await dispatchCreateUser(this.$store, updatedProfile);
    this.$router.push("/main/admin/users");
  }
}
</script>
