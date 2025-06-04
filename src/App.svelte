<script>
	import './app.css';
	import Notifications from './notifications.svelte';

	let notificationsComponent = $state();

	const sampleNotifications = [
		{
			id: 1,
			message: 'Welcome! Your account has been successfully created.',
			type: 'success'
		},
		{
			id: 2,
			message: 'New message received from John Doe.',
			type: 'info'
		},
		{
			id: 3,
			message: 'Your subscription will expire in 3 days.',
			type: 'warning'
		},
		{
			id: 4,
			message: 'Failed to save changes. Please try again.',
			type: 'error'
		},
		{
			id: 5,
			message: 'System maintenance scheduled for tonight at 2 AM.',
			type: 'info'
		},
		{
			id: 6,
			message: 'Are you sure you want to delete this item? This action cannot be undone.',
			type: 'confirm',
			confirmText: 'Delete',
			cancelText: 'Cancel',
			callback: (confirmed) => {
				if (confirmed) {
					notificationsComponent?.addNotification({
						id: Date.now(),
						message: 'Item deleted successfully!',
						type: 'success'
					});
				} else {
					notificationsComponent?.addNotification({
						id: Date.now(),
						message: 'Delete action cancelled.',
						type: 'info'
					});
				}
			}
		},
		{
			id: 7,
			message: 'Do you want to save your changes before leaving?',
			type: 'confirm',
			confirmText: 'Save & Leave',
			cancelText: 'Discard',
			callback: (confirmed) => {
				if (confirmed) {
					notificationsComponent?.addNotification({
						id: Date.now(),
						message: 'Changes saved successfully!',
						type: 'success'
					});
				} else {
					notificationsComponent?.addNotification({
						id: Date.now(),
						message: 'Changes discarded.',
						type: 'warning'
					});
				}
			}
		}
	];

	function addNotification() {
		const randomNotification =
			sampleNotifications[Math.floor(Math.random() * sampleNotifications.length)];

		const newNotification = {
			...randomNotification,
			id: Date.now() + Math.random()
		};

		notificationsComponent?.addNotification(newNotification);
	}

	function addMultipleNotifications() {
		sampleNotifications.forEach((notification, index) => {
			setTimeout(() => {
				const newNotification = {
					...notification,
					id: Date.now() + Math.random() + index
				};

				notificationsComponent?.addNotification(newNotification);
			}, index * 200);
		});
	}

	function addConfirmDialog() {
		const confirmNotifications = sampleNotifications.filter((n) => n.type === 'confirm');
		const randomConfirm =
			confirmNotifications[Math.floor(Math.random() * confirmNotifications.length)];

		const newNotification = {
			...randomConfirm,
			id: Date.now() + Math.random()
		};

		notificationsComponent?.addNotification(newNotification);
	}

	function addCustomConfirm() {
		notificationsComponent?.addNotification({
			id: Date.now(),
			message: 'This is a custom confirm dialog. Do you want to proceed with this action?',
			type: 'confirm',
			confirmText: 'Yes, Proceed',
			cancelText: 'No, Cancel',
			callback: (confirmed) => {
				const message = confirmed ? 'You confirmed the action!' : 'You cancelled the action.';
				const type = confirmed ? 'success' : 'info';

				notificationsComponent?.addNotification({ id: Date.now() + 1, message, type });
			}
		});
	}
</script>

<div class="min-h-screen bg-gray-50 p-8">
	<div class="mx-auto max-w-4xl">
		<h1 class="mb-8 text-3xl font-bold text-gray-900">Notifications Demo</h1>

		<div class="mb-8 flex flex-wrap space-y-2 space-x-4">
			<button
				onclick={addNotification}
				class="rounded-lg bg-blue-600 px-4 py-2 text-white transition-colors hover:bg-blue-700"
			>
				Add Random Notification
			</button>

			<button
				onclick={addMultipleNotifications}
				class="rounded-lg bg-green-600 px-4 py-2 text-white transition-colors hover:bg-green-700"
			>
				Add Multiple Notifications
			</button>

			<button
				onclick={addConfirmDialog}
				class="rounded-lg bg-purple-600 px-4 py-2 text-white transition-colors hover:bg-purple-700"
			>
				Add Confirm Dialog
			</button>

			<button
				onclick={addCustomConfirm}
				class="rounded-lg bg-orange-600 px-4 py-2 text-white transition-colors hover:bg-orange-700"
			>
				Add Custom Confirm
			</button>
		</div>

		<div class="rounded-lg bg-white p-6 shadow-lg">
			<h2 class="mb-4 text-xl font-semibold">Notification Area</h2>
			<p class="mb-4 text-gray-600">
				Click the buttons above to see notifications appear with typing animation.
			</p>

			<!-- Notifications will appear here -->
		</div>
	</div>

	<Notifications bind:this={notificationsComponent} />
</div>
