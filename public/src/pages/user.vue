<template>
	<div>
		<hmenu></hmenu>

		<div class="main">
			<h1>User</h1>

			<div class="box" v-if="addEditUser">
				<div class="wrapper">
					<h2>Create/Edit User</h2>

					<msg :type="'error'" :msg="'Login, E-Mail and password must be set.'" v-if="err == 1" v-on:close="closeMsg"></msg>
					<msg :type="'error'" :msg="'The passwords must be at least 8 characters long.'" v-if="err == 2" v-on:close="closeMsg"></msg>
					<msg :type="'error'" :msg="'The passwords must be equal.'" v-if="err == 3" v-on:close="closeMsg"></msg>
					<msg :type="'error'" :msg="'The Login and/or E-Mail is in use already.'" v-if="err == 4" v-on:close="closeMsg"></msg>
					<msg :type="'error'" :msg="'You have no permission to do this.'" v-if="err == 200" v-on:close="closeMsg"></msg>

					<form v-on:submit.prevent="performAddEditUser()">
						<table>
							<tr>
								<td class="w20">Login:</td>
								<td><input type="text" name="login" class="full-width" v-model="login" /></td>
							</tr>
							<tr>
								<td>E-Mail:</td>
								<td><input type="email" name="email" class="full-width" v-model="email" /></td>
							</tr>
							<tr>
								<td>Password:</td>
								<td><input type="password" name="pwd1" class="full-width" v-model="pwd1" /></td>
							</tr>
							<tr>
								<td>Repeat password:</td>
								<td><input type="password" name="pwd2" class="full-width" v-model="pwd2" /></td>
							</tr>
							<tr>
								<td>Administrator:</td>
								<td><input type="checkbox" name="admin" class="full-width" v-model="admin" /></td>
							</tr>
							<tr>
								<td>Moderator:</td>
								<td><input type="checkbox" name="moderator" class="full-width" v-model="moderator" /></td>
							</tr>
							<tr>
								<td></td>
								<td>
									<input type="submit" value="Save" />
									<button v-on:click.prevent="addEditUser = false">Cancel</button>
								</td>
							</tr>
						</table>
					</form>
				</div>
			</div>

			<div class="box" v-if="removeUser">
				<div class="wrapper">
					<h2>Remove User</h2>

					<msg :type="'error'" :msg="'You have no permission to do this.'" v-if="err == 200" v-on:close="closeMsg"></msg>

					<p>Do you really want to remove this user?</p>

					<button v-on:click="performRemoveUser()">Yes, remove user</button>
					<button v-on:click="removeUser = false">Cancel</button>
				</div>
			</div>

			<div class="box">
				<div class="wrapper">
					<msg :type="'success'" :msg="'The user has been saved.'" v-if="userSaved" v-on:close="userSaved = false"></msg>
					<msg :type="'success'" :msg="'The user has been removed.'" v-if="userRemoved" v-on:close="userRemoved = false"></msg>

					<button v-on:click="openAddEditUser(0)">Add User</button>

					<table>
						<thead>
							<tr>
								<td class="w5">ID</td>
								<td class="w30">Login</td>
								<td class="w30">E-Mail</td>
								<td class="w30">Role</td>
								<td></td>
							</tr>
						</thead>
						<tbody>
							<tr v-for="u in user">
								<td>{{u.id}}</td>
								<td>{{u.login}}</td>
								<td>{{u.email}}</td>
								<td>
									<span v-if="u.admin">Administrator</span>
									<span v-if="u.moderator">Moderator</span>
								</td>
								<td>
									<i class="fa fa-pencil" aria-hidden="true" title="Edit user" v-on:click="openAddEditUser(u.id)"></i>
									<i class="fa fa-trash" aria-hidden="true" title="Remove user" v-on:click="openRemoveUser(u.id)"></i>
								</td>
							</tr>
						</tbody>
					</table>
				</div>
			</div>

			<div class="box">
				<div class="wrapper">
					<strong>Administrators</strong> have full access and can change all settings (including changing user passwords).<br />
					<strong>Moderators</strong> can start/stop server instances and create, change and remove configurations.<br />
					Other users have read access, except for user E-Mail addresses.
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import axios from "axios";
import {hmenu, msg} from "../components";

export default {
	components: {
		hmenu,
		msg
	},
	data() {
		return {
			user: [],
			_id: 0,
			login: '',
			email: '',
			pwd1: '',
			pwd2: '',
			admin: false,
			moderator: false,
			err: 0,
			addEditUser: false,
			removeUser: false,
			userSaved: false,
			userRemoved: false
		}
	},
	mounted() {
		this._load();
	},
	methods: {
		_load() {
			axios.get('/api/user')
			.then(resp => {
				if(resp.data.code){
					console.log(resp.data.code+': '+resp.data.msg);
					return;
				}

				this.user = resp.data;
			});
		},
		_reset() {
			this._id = 0;
			this.login = '';
			this.email = '';
			this.pwd1 = '';
			this.pwd2 = '';
			this.admin = false;
			this.moderator = false;
			this.err = 0;
			this.addEditUser = false;
			this.removeUser = false;
			this.userSaved = false;
			this.userRemoved = false;
		},
		openAddEditUser(id) {
			this._reset();
			
			if(id){
				this._id = id;

				axios.get('/api/user', {params: {id: id}})
				.then(resp => {
					if(resp.data.code){
						console.log(resp.data.code+': '+resp.data.msg);
						return;
					}

					this.login = resp.data.login;
					this.email = resp.data.email;
					this.admin = resp.data.admin;
					this.moderator = resp.data.moderator;
					this.addEditUser = true;
				});
			}
			else{
				this.addEditUser = true;
			}
		},
		openRemoveUser(id) {
			this._reset();

			if(!id){
				return;
			}

			this._id = id;
			this.removeUser = true;
		},
		performAddEditUser() {
			axios.post('/api/user', {id: this._id,
				login: this.login,
				email: this.email,
				pwd1: this.pwd1,
				pwd2: this.pwd2,
				admin: this.admin,
				moderator: this.moderator})
			.then(resp => {
				if(resp.data.code){
					console.log(resp.data.code+': '+resp.data.msg);
					this.err = resp.data.code;
					return;
				}

				this._reset();
				this._load();
				this.userSaved = true;
			});
		},
		performRemoveUser() {
			axios.delete('/api/user', {params: {id: this._id}})
			.then(resp => {
				if(resp.data.code){
					console.log(resp.data.code+': '+resp.data.msg);
					this.err = resp.data.code;
					return;
				}

				this._reset();
				this._load();
				this.userRemoved = true;
			});
		},
		closeMsg() {
			this.err = 0;
		}
	}
};
</script>

<style lang="scss">
</style>
