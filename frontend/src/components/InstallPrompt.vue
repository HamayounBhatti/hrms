<template>
	<!-- Install PWA dialog -->
	<Dialog v-model="showDialog">
		<template #body-title>
			<h2 class="text-lg font-bold">{{ __("Install Xconsol HR") }} </h2>
		</template>
		<template #body-content>
			<p>{{ __("Get the app on your device for easy access & a better experience!") }} </p>
		</template>
		<template #actions>
			<Button variant="solid" @click="() => install()" class="py-5 w-full">
				<template #prefix><FeatherIcon name="download" class="w-4" /></template>
				{{ __("Install") }}
			</Button>
		</template>
	</Dialog>

	<!-- iOS installation info message -->
	<Popover :show="iosInstallMessage" placement="bottom">
		<template #body>
			<div
				class="mt-[calc(100vh-15rem)] flex flex-col gap-3 mx-2 rounded py-5 bg-blue-100 drop-shadow-xl"
			>
				<div
					class="flex flex-row text-center items-center justify-between mb-1 px-3"
				>
					<span class="text-base text-gray-900 font-bold">
						{{ __("Install Xconsol HR") }}
					</span>
					<span class="inline-flex items-baseline">
						<FeatherIcon
							name="x"
							class="ms-auto h-4 w-4 text-gray-700"
							@click="iosInstallMessage = false"
						/>
					</span>
				</div>
				<div class="text-xs text-gray-800 px-3">
					<span class="flex flex-col gap-2">
						<span>
							{{ __("Get the app on your iPhone for easy access & a better experience") }}
						</span>
						<span class="inline-flex items-start whitespace-nowrap">
							<span>{{ __("Tap") }}&nbsp;</span>
							<FeatherIcon name="share" class="h-4 w-4 text-blue-600" />
							<span>&nbsp;{{ __('and then "Add to Home Screen"') }}</span>
						</span>
					</span>
				</div>
			</div>
		</template>
	</Popover>
</template>

<script setup>
import { ref, watch } from "vue"

import { Dialog, Popover, FeatherIcon } from "frappe-ui"

const DISMISS_KEY = "xconsol_install_prompt_dismissed"

// Initialize deferredPrompt for use later to show browser install prompt.
const deferredPrompt = ref(null)
const showDialog = ref(false)
const iosInstallMessage = ref(false)

const isIos = () => {
	// Detects if device is on iOS
	const userAgent = window.navigator.userAgent.toLowerCase()
	return /iphone|ipad|ipod/.test(userAgent)
}

// Detects if device is in standalone mode
const isInStandaloneMode = () =>
	"standalone" in window.navigator && window.navigator.standalone

const wasPromptDismissed = () => {
	try {
		return window.localStorage.getItem(DISMISS_KEY) === "1"
	} catch {
		return false
	}
}

const markPromptDismissed = () => {
	try {
		window.localStorage.setItem(DISMISS_KEY, "1")
	} catch {
		// ignore storage errors
	}
}

// Show the install prompt only once. Once the user dismisses it or installs
// the app, remember it and never nag them again.
if (!wasPromptDismissed() && isIos() && !isInStandaloneMode()) {
	iosInstallMessage.value = true
}

window.addEventListener("beforeinstallprompt", (e) => {
	// Prevent the mini-infobar from appearing on mobile
	e.preventDefault()
	// Stash the event so it can be triggered later.
	deferredPrompt.value = e

	if (wasPromptDismissed() || isInStandaloneMode()) {
		return
	}

	if (isIos()) {
		iosInstallMessage.value = true
	} else {
		showDialog.value = true
	}
})

window.addEventListener("appinstalled", () => {
	showDialog.value = false
	deferredPrompt.value = null
	markPromptDismissed()
})

watch(showDialog, (value, oldValue) => {
	if (oldValue && !value) {
		markPromptDismissed()
	}
})

watch(iosInstallMessage, (value, oldValue) => {
	if (oldValue && !value) {
		markPromptDismissed()
	}
})

async function install() {
	if (!deferredPrompt.value) return
	deferredPrompt.value.prompt()
	showDialog.value = false
	markPromptDismissed()
}
</script>
