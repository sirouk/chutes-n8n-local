<script setup lang="ts">
import { N8nFormBox, N8nLogo, N8nText } from '@n8n/design-system';

import SSOLogin from '@/features/settings/sso/components/SSOLogin.vue';
import type { FormFieldValueUpdate, IFormBoxConfig } from '@/Interface';
import { useSettingsStore } from '@/app/stores/settings.store';
import type { EmailOrLdapLoginIdAndPassword } from './SigninView.vue';

withDefaults(
	defineProps<{
		form: IFormBoxConfig;
		formLoading?: boolean;
		subtitle?: string;
		withSso?: boolean;
		showPasswordLogin?: boolean;
		passwordLoginToggleLabel?: string;
	}>(),
	{
		formLoading: false,
		withSso: false,
		showPasswordLogin: true,
		passwordLoginToggleLabel: '',
	},
);

const emit = defineEmits<{
	update: [FormFieldValueUpdate];
	submit: [values: EmailOrLdapLoginIdAndPassword];
	secondaryClick: [];
	togglePasswordLogin: [];
}>();

const onUpdate = (e: FormFieldValueUpdate) => {
	emit('update', e);
};

const onSubmit = (data: unknown) => {
	emit('submit', data as EmailOrLdapLoginIdAndPassword);
};

const onSecondaryClick = () => {
	emit('secondaryClick');
};

const onTogglePasswordLogin = () => {
	emit('togglePasswordLogin');
};

const {
	settings: { releaseChannel },
} = useSettingsStore();
</script>

<template>
	<div :class="$style.container">
		<N8nLogo size="large" :release-channel="releaseChannel" />
		<div v-if="subtitle" :class="$style.textContainer">
			<N8nText size="large">{{ subtitle }}</N8nText>
		</div>
		<SSOLogin v-if="withSso" :class="$style.primarySso" />
		<div v-if="!withSso || showPasswordLogin" :class="$style.formContainer">
			<N8nFormBox
				v-bind="form"
				data-test-id="auth-form"
				:button-loading="formLoading"
				@secondary-click="onSecondaryClick"
				@submit="onSubmit"
				@update="onUpdate"
			/>
		</div>
		<div v-else-if="passwordLoginToggleLabel" :class="$style.passwordToggleContainer">
			<button
				type="button"
				data-test-id="toggle-password-login"
				:class="$style.passwordLoginToggle"
				@click="onTogglePasswordLogin"
			>
				{{ passwordLoginToggleLabel }}
			</button>
		</div>
	</div>
</template>

<style lang="scss" module>
body {
	background-color: var(--color--background--light-2);
}

.container {
	display: flex;
	align-items: center;
	flex-direction: column;
	padding-top: var(--spacing--2xl);

	> * {
		width: 352px;
	}
}

.textContainer {
	text-align: center;
}

.primarySso {
	margin-top: var(--spacing--l);
}

.formContainer {
	padding-bottom: var(--spacing--xl);
}

.passwordToggleContainer {
	padding-bottom: var(--spacing--xl);
	text-align: center;
}

.passwordLoginToggle {
	cursor: pointer;
	border: none;
	background: transparent;
	padding: 0;
	color: var(--color-text-base);
	font: inherit;
	text-decoration: underline;

	&:hover,
	&:focus-visible {
		color: var(--color-primary);
	}
}
</style>

<style lang="scss">
.el-checkbox__label span {
	font-size: var(--font-size-2xs) !important;
}
</style>
