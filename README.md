import React, { useState } from 'react';
import Schema from '../../schema/AppSchema';
import logo from '../../assets/images/logo.png';
import userLogo from '../../assets/images/user.png';
import {
	Select,
	SelectOption,
	List,
	ListItemText,
	Tooltip,
	Input,
	Button,
	Avatar,
} from 'appkit-react';
import FDialog from '../dialog/Dialog';
import UpdatePasswordDialog from './UpdatePasswordDialog';
import { useMutation } from '@apollo/react-hooks';
import * as FormValidation from '../../Validation';
import { Growl } from 'primereact/growl';
import './Header.scss';

const DATE_OPTIONS = {
	year: 'numeric',
	month: 'short',
	day: 'numeric',
	hour: '2-digit',
	minute: '2-digit',
};
const date = new Date().toLocaleDateString('en-US', DATE_OPTIONS);

const UPDATE_PROFILE = Schema.updateOneUser;
const UPLOAD_IMAGE = Schema.singleUpload;
const UPDATE_PASSWORD = Schema.updatePassword;


const logout = (props) => {
	localStorage.clear();
	props.history.push('/login');
};

const FModalHeader = () => {
	return <div>Edit Profile</div>;
};

const FModalHeaderUpdatePw = () => {
	return <div>Change Password</div>;
};

const Header = (props) => {
	let profileForm = {};
	let userInfo = JSON.parse(localStorage.getItem('USER_INF'));
	const [showProfile, setShowProfile] = useState(false);
	const [showUpdatePw, setShowUpdatePw] = useState(false);

	const [growl, setGrowl] = useState();
	const [profileState, setProfileState] = useState({
		first_name: userInfo?.first_name,
		last_name: userInfo?.last_name,
		// new_password_Icon: true,
		// confirm_new_password_Icon: true,
	});
	const [statePassword, setStatePassword] = useState({
		new_password: '',
		confirm_new_password: '',
		new_password_Icon: true,
		confirm_new_password_Icon: true,
	});

	const [errors, setErrors] = useState(false);
	const [email] = useState(userInfo?.email);
	const [role] = useState(userInfo?.role['name']);

	const [updateProfile, { data, loading, called }] = useMutation(
		UPDATE_PROFILE,
		{
			onCompleted: (response) => {
				userInfo = response.updateOneUser;
				setProfileState({
					first_name: userInfo?.first_name,
					last_name: userInfo?.last_name,
					img_url: userInfo.img_url,
				});
				if (response) {
					growl.show({
						severity: 'success',
						summary: 'Profile updated sucessfully',
					});
				}
			},
			onError: () => { },
		}
	);

	const [updatePassword] = useMutation(UPDATE_PASSWORD, {
		onCompleted: (response) => {
			if (response) {
				growl.show({
					severity: 'success',
					summary: 'Password updated sucessfully',
				});
			}
		},
	});

	const [uploadImage] = useMutation(UPLOAD_IMAGE);
	const openProfile = () => {
		setShowProfile(true);
	};
	const openUpdatePassword = () => {
		setShowUpdatePw(true);
	};
	const logoutTooltip = (dProps) => (
		<div>
			<List>
				{/* <ListItemText onClick={() => onClickMenu(dProps, 'users')}><span className="appkiticon secondary-color icon-avatar-outline m-r-10" />Users</ListItemText>
				<ListItemText onClick={() => onClickMenu(dProps, 'roles')}><span className="appkiticon secondary-color icon-avatar-outline m-r-10" />Roles</ListItemText> */}
				<ListItemText onClick={() => openProfile(dProps)}>
					<span className="appkiticon secondary-color icon-avatar-outline m-r-10" />
          Profile
        </ListItemText>
				<ListItemText onClick={() => openUpdatePassword(dProps)}>
					<span className="change_pw change_pw_secondry pi pi-key " />
          Change Password{' '}
				</ListItemText>
				<ListItemText onClick={() => logout(dProps)}>
					<span className="appkiticon secondary-color icon-log-out-outline m-r-10" />
          Logout
        </ListItemText>
			</List>
		</div>
	);
	const handleCancel = () => {
		setShowProfile(false);
		setErrors(false);
		setProfileState({
			...profileState,
			first_name: userInfo?.first_name,
			last_name: userInfo?.last_name,
		});
	};
	const handleCancelUpdatePW = () => {
		setShowUpdatePw(false);
		setErrors(false);
		setStatePassword({
			...statePassword,
			new_password: '',
			confirm_new_password: '',
			new_password_Icon: true,
			confirm_new_password_Icon: true,
		});
		// setProfileState({
		// 	...profileState,
		// 	new_password_Icon: true,
		// 	confirm_new_password_Icon: true,
		// });
	};

	const handleChange = (ev, key) => {
		const value = ev;
		setProfileState({
			...profileState,
			[key]: value,
		});
		validateFeild(ev, key);
	};
	const handleChangePassword = (ev, key) => {
		const value = ev;
		setStatePassword({
			...statePassword,
			[key]: value,
		});
		validateFeild(ev, key);
	};

	const validateFeild = async (ev, key) => {
		let err = {};
		if (key === 'first_name') {
			err = FormValidation.validateFileld('required', key, ev);
		} else if (key === 'last_name') {
			err = FormValidation.validateFileld('required', key, ev);
		} else if (key === 'new_password') {
			err = FormValidation.validateFileld('required', key, ev);
		} else if (key === 'confirm_new_password') {
			err = FormValidation.validateFileld('required', key, ev);
		}
		let prevErrors = { ...errors, ...err };
		setErrors(prevErrors);
	};

	const getImageName = () => {
		let name = '';
		if (userInfo?.first_name && userInfo?.first_name.length > 0) {
			name = name + userInfo?.first_name[0].toUpperCase();
		}
		if (userInfo?.last_name && userInfo?.last_name.length > 0) {
			name = name + userInfo?.last_name[0].toUpperCase();
		}
		return name;
	};
	const [fileUploader, setFileUploader] = useState('');
	const handleUploadImage = () => {
		fileUploader.click();
	};

	const [uploadedImg, setUploadedImg] = useState('');
	const handleImageUpload = (ev) => {
		ev.stopPropagation();
		ev.preventDefault();
		var file = ev.target.files[0];
		if (file) {
			setUploadedImg();
			// let uploadData = new FormData();
			// uploadData.append('file', file);
			uploadImage({
				variables: {
					file: file,
				},
			});
		}
	};
	const handlerNewPassword = (e) => {
		e.preventDefault();
		setStatePassword({
			...statePassword,
			new_password_Icon: !statePassword.new_password_Icon,
		});
	};
	const handlerConfirmNewPassword = (e) => {
		e.preventDefault();
		setStatePassword({
			...statePassword,
			confirm_new_password_Icon: !statePassword.confirm_new_password_Icon,
		});
	};

	const FModalBody = () => {
		return (
			<div className="row">
				<form ref={(form) => (profileForm = form)} className="w-100">
					<div className="col-xs-12 col-sm-12 col-lg-12 col-sm-12 col-xl-12 m-t-20 text-center">
						<Avatar
							className="profile-img"
							onClick={() => handleUploadImage()}
							size="xlarge"
							shape="circle"
							src={userInfo?.img_url || uploadedImg}
							label={getImageName()}
						></Avatar>
						<input
							type="file"
							id="file"
							ref={(file) => {
								setFileUploader(file);
							}}
							hidden
							onChange={(ev) => handleImageUpload(ev)}
						/>
					</div>
					<div className="col-xs-12 col-sm-12 col-lg-12 col-sm-12 col-xl-12 m-t-20">
						<label>First Name</label>
						<Input
							inputBoxSize="sm"
							name="first_name"
							type="text"
							value={profileState.first_name}
							validations={['required']}
							errMsg="First Name is mandatory"
							hasError={errors && errors.first_name}
							maxLength={15}
							onChange={(e) => handleChange(e, 'first_name')}
						/>
					</div>
					<div className="col-xs-12 col-sm-12 col-lg-12 col-sm-12 col-xl-12 m-t-20">
						<label>Last Name</label>
						<Input
							inputBoxSize="sm"
							name="last_name"
							type="text"
							value={profileState.last_name}
							validations={['required']}
							errMsg="Last Name is mandatory"
							hasError={errors && errors.last_name}
							maxLength={15}
							onChange={(e) => handleChange(e, 'last_name')}
						/>
					</div>
					<div className="col-xs-12 col-sm-12 col-lg-12 col-sm-12 col-xl-12 m-t-20">
						<label>Email</label>
						<Input
							inputBoxSize="sm"
							key="email"
							value={email}
							disabled={true}
						/>
					</div>
					<div className="col-xs-12 col-sm-12 col-lg-12 col-sm-12 col-xl-12 m-t-20">
						<label>Roles</label>
						<Input inputBoxSize="sm" key="email" value={role} disabled={true} />
					</div>
				</form>
			</div>
		);
	};

	const FModalBodyUpdatePW = () => {
		return (
			<div className="row">
				<form ref={(form) => (profileForm = form)} className="w-100">
					<div className="col-xs-12 col-sm-12 col-lg-12 col-sm-12 col-xl-12 m-t-20">
						<label>New Password</label>
						<Input
							inputBoxSize="sm"
							name="new_password"
							type={statePassword.new_password_Icon ? 'password' : 'text'}
							value={statePassword.new_password}
							validations={['required']}
							errMsg="New Password is mandatory"
							hasError={errors && errors.new_password}
							maxLength={25}
							onChange={(e) => handleChangePassword(e, 'new_password')}
						/>
						<span
							onClick={(e) => handlerNewPassword(e, 'new_password')}
							className={
								statePassword.new_password_Icon
									? 'btn-show_password a-font-22 pi pi-eye-slash'
									: 'btn-show_password a-font-22 pi pi-eye'
							}
						/>
					</div>
					<div className="col-xs-12 col-sm-12 col-lg-12 col-sm-12 col-xl-12 m-t-20">
						<label>Confirm New Password</label>
						<Input
							inputBoxSize="sm"
							name="confirm_new_password"
							type={
								statePassword.confirm_new_password_Icon ? 'password' : 'text'
							}
							value={statePassword.confirm_new_password}
							validations={['required']}
							errMsg="Confirm New Password is mandatory"
							hasError={errors && errors.confirm_new_password}
							maxLength={25}
							onChange={(e) => handleChangePassword(e, 'confirm_new_password')}
						/>
						<span
							onClick={(e) =>
								handlerConfirmNewPassword(e, 'confirm_new_password')
							}
							className={
								statePassword.confirm_new_password_Icon
									? 'btn-show_password a-font-22 pi pi-eye-slash'
									: 'btn-show_password a-font-22 pi pi-eye'
							}
						/>
					</div>
				</form>
			</div>
		);
	};

	const FModalFooter = () => {
		return (
			<div className="button-comp">
				<Button
					size="md"
					kind="primary"
					className="m-r-20 w-74 f-wt-bold"
					onClick={handleSave}
				>
					{' '}
          SAVE{' '}
				</Button>
				<Button
					size="md"
					kind=""
					className="outline-btn w-74"
					onClick={handleCancel}
				>
					{' '}
          CANCEL{' '}
				</Button>
			</div>
		);
	};

	const FModalFooterUpdatePW = () => {
		return (
			<div className="button-comp">
				<Button
					size="md"
					kind="primary"
					className="m-r-20 w-74 f-wt-bold"
					onClick={handleSavePW}
				>
					{' '}
          SAVE{' '}
				</Button>
				<Button
					size="md"
					kind=""
					className="outline-btn w-74"
					onClick={handleCancelUpdatePW}
				>
					{' '}
          CANCEL{' '}
				</Button>
			</div>
		);
	};

	const handleSave = () => {
		let formErrors = FormValidation.validateForm(profileForm);
		var exists = Object.keys(formErrors).some(function (k) {
			return formErrors[k] === true;
		});
		if (exists) {
			alert('All are mandatory');
			return;
		}

		updateProfile({
			variables: {
				where: { id: userInfo.id },
				data: {
					first_name: profileState.first_name,
					last_name: profileState.last_name,
				},
			},
		});
		handleCancel();
	};

	const handleSavePW = () => {
		let formErrors = FormValidation.validateForm(profileForm);
		var exists = Object.keys(formErrors).some(function (k) {
			return formErrors[k] === true;
		});

		if (exists) {
			alert('All are mandatory');
			return;
		}

		if (statePassword.confirm_new_password === statePassword.new_password) {
			if (statePassword.confirm_new_password.length > 0) {
				updatePassword({
					variables: {
						id: userInfo.id,
						password: statePassword.new_password,
					},
				});
				handleCancelUpdatePW();
			} else {
				growl.show({
					severity: 'error',
					sticky: true,
					summary: 'All fields are mandatory',
				});
			}
		} else {
			growl.show({
				severity: 'error',
				sticky: true,
				summary:
					'New password and Confirm new password values both should be same',
			});
		}
	};

	if (called) {
		if (!loading) {
			localStorage.setItem('USER_INF', JSON.stringify(data.updateOneUser));
			userInfo = JSON.parse(localStorage.getItem('USER_INF'));
		}
	}
	const onClickLogo = () => {
		props.history.push('/dashboard');
	};

	return (
		<div className="normal-header-container">
			{showProfile && (
				<FDialog
					showModal={showProfile}
					handleCancel={handleCancel}
					FModalHeader={FModalHeader}
					FModalBody={FModalBody}
					FModalFooter={FModalFooter}
					className="common_css UPDATE_PROFILE"
				/>
			)}
			{showUpdatePw && (
				<UpdatePasswordDialog
					showModal={showUpdatePw}
					handleCancelUpdatePW={handleCancelUpdatePW}
					FModalHeaderUpdatePW={FModalHeaderUpdatePw}
					FModalBodyUpdatePW={FModalBodyUpdatePW}
					FModalFooterUpdatePW={FModalFooterUpdatePW}
					className="common_css UPDATE_PROFILE"
				/>
			)}
			<Growl ref={(el) => setGrowl(el)} />
			<nav className={'a-header-wrapper justify-content-between'}>
				<div className="a-brand-container">
					<div className="pointer">
						<img src={logo} height="30" alt="Tax Payer" onClick={onClickLogo} />
					</div>
					<div className="a-divider" />
					<h6 className="text-nowrap m-b-0 fs-18">
						<span className="h1-name">Amritsar Municipal Corporation</span>
						<span className="h1-name2">
							- Financial Management Information System
            </span>
					</h6>
				</div>
				<div className="a-actions-container">
					<button className="a-btn-background-transparent d-wi-none m-r-20 a-icon-container">
						<span className="a-font-22 appkiticon icon-primary icon-archive-fill" />
					</button>
					<button className="a-btn-background-transparent d-wi-none  m-r-20 a-icon-container">
						<span className="a-font-22 appkiticon icon-primary icon-grow-revenue-fill" />
					</button>
					<button className="a-btn-background-transparent d-wi-none m-r-20 a-icon-container">
						<span className="a-font-22 appkiticon icon-primary icon-add-user-fill" />
					</button>
					<button className="a-btn-background-transparent d-wi-none m-r-20 a-icon-container">
						<span className="a-font-22 appkiticon icon-primary icon-help-question-fill" />
					</button>
					<button className="a-btn-background-transparent d-wi-none m-r-20 a-icon-container">
						<span className="a-font-22 appkiticon icon-primary icon-notification-fill" />
					</button>

					<div className="p-r-20 text-right h1-name2 d-wi-none flex-d-c">
						<div>
							{userInfo?.first_name} {userInfo?.last_name}
						</div>
						<div className="ellipses">
							Last Login: <span>{date}</span>
						</div>
					</div>
					<Tooltip
						content={logoutTooltip(props)}
						trigger="click"
						refClassName="a-account-info"
						className="a-account-options"
						tooltipTheme="light"
						clickToClose
						placement="bottom-end"
					>
						<img
							className="pointer"
							src={userLogo}
							height="30"
							alt="Tax Payer"
						/>
					</Tooltip>
					<div className="d-wi a-sm-options-container">
						<Select value="1">
							<SelectOption
								key="1"
								value="1"
								selectedDisplay={
									<span className="appkiticon icon-vertical-more-fill" />
								}
							>
								<span className="a-font-18 appkiticon icon-primary icon-archive-fill" />
								<span className="a-description">Bookmarks</span>
							</SelectOption>
							<SelectOption
								key="2"
								value="2"
								selectedDisplay={
									<span className="appkiticon icon-vertical-more-fill" />
								}
							>
								<span className="a-font-18 appkiticon icon-primary icon-grow-revenue-fill" />
								<span className="a-description">Graph View</span>
							</SelectOption>
							<SelectOption
								key="3"
								value="3"
								selectedDisplay={
									<span className="appkiticon icon-vertical-more-fill" />
								}
							>
								<span className="a-font-18 appkiticon icon-primary icon-add-user-fill" />
								<span className="a-description">Performance</span>
							</SelectOption>
							<SelectOption
								key="4"
								value="4"
								selectedDisplay={
									<span className="appkiticon icon-vertical-more-fill" />
								}
							>
								<span className="a-font-18 appkiticon icon-primary icon-help-question-fill" />
								<span className="a-description">Help</span>
							</SelectOption>
							<SelectOption
								key="5"
								value="5"
								selectedDisplay={
									<span className="appkiticon icon-vertical-more-fill" />
								}
							>
								<span className="a-font-18 appkiticon icon-primary icon-notification-fill" />
								<span className="a-description">Alerts</span>
							</SelectOption>
						</Select>
					</div>
				</div>
			</nav>
		</div>
	);
};
export default Header;
