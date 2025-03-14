<template>
    <journal-layout>
        <ContentUpdateNotification v-if="contentUpdateAvailable" :reloading="reloading" @reload="reload">
            Collection updated
        </ContentUpdateNotification>

        <div v-if="$page.props.collections.length === 1 && collection.bullets.length === 0" class="mb-10 leading-relaxed text-gray-600 dark:text-gray-300">
            <h1 class="text-xl font-semibold text-gray-700 dark:text-gray-300">
                <Icon name="medium/clipboard" auto-size class="mr-1 text-gray-600 dark:text-gray-400" />
                Collections
            </h1>
            <p class="mt-4">Collections can be used for all sorts of things, such as:</p>
            <ul class="mt-4 list-disc ml-4">
                <li>Project ideas</li>
                <li>Books to read</li>
                <li>Movies and TV shows to watch</li>
                <li>Shopping lists</li>
                <li>A menu of things you like to eat</li>
                <li>A meal plan</li>
            </ul>
            <p class="mt-4">Collections can also be shared with other users. Handy for a family shopping list!</p>
            <p class="mt-4">You can also choose between a bulleted list, or a simple checklist. Also toggle between showing or hiding your completed items.</p>
            <p class="mt-4">Unlike the daily log, nothing will fade away here, so try not to let things get out of hand.</p>
            <p class="mt-4">Enter your first item to get started...</p>
        </div>

        <div class="flex flex-col">
            <div class="-mt-3 flex items-center justify-between" :class="drawer ? '' : 'border-b border-gray-200 dark:border-gray-700'">
                <input
                    type="text"
                    v-model.lazy="name"
                    class="w-full flex-1 px-0 py-3 font-bold border-none bg-transparent text-gray-700 dark:text-gray-200 focus:ring-0"
                    spellcheck="false"
                />

                <div class="flex flex-shrink-0 gap-2 text-gray-400">
                    <button
                        type="button"
                        @click="hideDone = ! hideDone"
                        class="p-2 rounded-md hover:bg-gray-100 dark:hover:bg-gray-900 focus:outline-none focus:bg-gray-100 dark:focus:bg-gray-900 transition duration-150 ease-in-out"
                        :class="hideDone ? 'text-gray-400 hover:text-gray-500 focus:text-gray-500' : 'text-pink-600 hover:text-pink-700 focus:text-pink-700'"
                        title="Show done"
                    >
                        <Icon name="medium/clipboard-check" class="w-6 h-6 md:w-5 md:h-5" />
                    </button>

                    <button
                        type="button"
                        @click="drawer = drawer === 'share' ? '' : 'share'"
                        class="p-2 rounded-md hover:bg-gray-100 dark:hover:bg-gray-900 focus:outline-none focus:bg-gray-100 dark:focus:bg-gray-900 transition duration-150 ease-in-out"
                        :class="drawer === 'share' ? 'bg-gray-100 dark:bg-gray-900 text-gray-500 dark:text-gray-300' : 'text-gray-400'"
                    >
                        <Icon name="medium/share" class="w-6 h-6 md:w-5 md:h-5" />
                    </button>

                    <jet-dropdown>
                        <template #trigger>
                            <button
                                type="button"
                                class="p-2 rounded-md hover:bg-gray-100 dark:hover:bg-gray-900 focus:outline-none focus:bg-gray-100 dark:focus:bg-gray-900 transition duration-150 ease-in-out text-gray-400"
                            >
                                <Icon name="medium/adjustments" class="w-6 h-6 md:w-5 md:h-5" />
                            </button>
                        </template>
                        <template #content>
                            <div class="px-4 py-3 space-y-2">
                                <div class="text-xs text-gray-400 dark:text-gray-300">
                                    List style
                                </div>
                                <label class="flex items-center">
                                    <input type="radio" v-model="type" name="type" value="bullet" class="h-5 w-5 border-gray-300 dark:border-gray-600 dark:bg-gray-800 dark:text-gray-700">
                                    <span class="ml-2 font-semibold text-gray-700 dark:text-gray-300">Bullets</span>
                                </label>
                                <label class="flex items-center">
                                    <input type="radio" v-model="type" name="type" value="checklist" class="h-5 w-5 border-gray-300 dark:border-gray-600 dark:bg-gray-800 dark:text-gray-700">
                                    <span class="ml-2 font-semibold text-gray-700 dark:text-gray-300">Checklist</span>
                                </label>
                            </div>
                            <div class="border-t border-gray-100 dark:border-gray-600"></div>
                            <jet-dropdown-link as="button" @click.native="confirmingClearDone = true">
                                <svg class="-mt-px mr-1 h-4 w-4 inline-block text-gray-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 13h6m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
                                </svg>
                                Clear Done
                            </jet-dropdown-link>
                            <jet-dropdown-link as="button" @click.native="confirmingDeleteCollection = true">
                                <svg class="-mt-px mr-1 h-4 w-4 inline-block text-gray-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                                </svg>
                                Delete Collection
                            </jet-dropdown-link>
                        </template>
                    </jet-dropdown>
                </div>
            </div>

            <div
                v-show="drawer"
                class="-mx-4 px-4 sm:-mx-6 sm:px-6 md:-mx-12 md:px-12 py-6 bg-gray-100 dark:bg-black dark:bg-opacity-20 shadow-inner"
            >
                <div v-show="drawer === 'share'" class="grid gap-6">
                    <div v-if="collection.users.length > 0" class="p-6 space-y-6 border border-gray-300 dark:border-gray-700 rounded">
                        <div v-for="user in collection.users" :key="user.id" class="flex items-center justify-between">
                            <div class="flex items-center">
                                <img class="w-8 h-8 rounded-full" :src="user.profile_photo_url" :alt="user.name">
                                <div class="ml-4 text-gray-900 dark:text-gray-100">{{ user.name }}</div>
                            </div>

                            <div class="flex items-center">
                                <button class="cursor-pointer ml-6 text-sm text-red-500 focus:outline-none" @click="confirmUserRemoval(user)">
                                    Remove
                                </button>
                            </div>
                        </div>
                    </div>

                    <form @submit.prevent="addUserForm.post(route('c.users.store', collection.hashid))">
                        <h3 class="text-gray-800 dark:text-gray-200">
                            Add a new collaborator, allowing them to view and edit this collection.
                        </h3>

                        <div class="mt-2 max-w-xl text-sm text-gray-600 dark:text-gray-400">
                            Please provide the email address of the person you would like to add to this collection. The email address must be associated with an existing account.
                        </div>

                        <label for="email" class="mt-4 block font-semibold text-gray-600 dark:text-gray-400">Email</label>

                        <div class="mt-1 flex flex-wrap justify-end gap-4">
                            <input
                                id="email"
                                type="email"
                                name="email"
                                class="flex-1 block border-gray-300 dark:border-gray-500 rounded bg-transparent text-gray-900 dark:text-gray-100"
                                v-model="addUserForm.email"
                            />

                            <button
                                type="submit"
                                class="px-4 py-2 flex items-center border border-gray-300 dark:border-gray-500 rounded font-medium text-gray-900 dark:text-gray-100"
                                :disabled="addUserForm.processing"
                            >
                                <Icon name="small/user-add" class="mr-1 text-gray-400" style="height: 1em; width: 1em;" />
                                Add
                            </button>
                        </div>

                        <jet-input-error for="email" :message="addUserForm.errors.email" class="mt-2" />
                    </form>
                </div>
            </div>
        </div>

        <draggable
            v-model="collection.bullets"
            handle=".drag-handle"
            @update="saveOrder"
        >
            <bullet
                v-for="bullet in collection.bullets"
                v-show="! bullet.complete || ! hideDone"
                :key="bullet.id + bullet.updated_at"
                :bullet="bullet"
                :type="type"
                :draggable="true"
                @input="updateBullet"
                @delete="deleteBullet"
                @migrateTo="migrateBulletTo"
                @migrateToDailyLog="migrateBulletToDailyLog"
            >
                <template #status>
                    <Link
                        v-if="bullet.date && bullet.user_id === $page.props.user.id"
                        :href="route('daily-log.index')"
                        title="Appears in daily log"
                        class="inline-block md:-mt-1 -mb-1 p-2 rounded-md text-gray-400 dark:text-gray-500 hover:bg-gray-100 dark:hover:bg-gray-900 focus:outline-none focus:bg-gray-100 dark:focus:bg-gray-900 transition duration-150 ease-in-out"
                        :class="[
                            bullet.complete ? 'opacity-50' : ''
                        ]"
                    >
                        <Icon name="medium/calendar" class="h-6 w-6 md:h-5 md:w-5" />
                    </Link>
                </template>
            </bullet>
        </draggable>

        <new-bullet @input="storeBullet" />

        <jet-confirmation-modal :show="confirmingClearDone" @close="confirmingClearDone = false">
            <template #title>
                Clear Done
            </template>
            <template #content>
                Are you sure you want to clear all complete items?
            </template>
            <template #footer>
                <jet-secondary-button @click.native="confirmingClearDone = false">
                    Nevermind
                </jet-secondary-button>

                <jet-danger-button class="ml-2" @click.native="clearDone" :class="{ 'opacity-25': processing }" :disabled="processing">
                    Clear Done
                </jet-danger-button>
            </template>
        </jet-confirmation-modal>

        <jet-confirmation-modal :show="confirmingDeleteCollection" @close="confirmingDeleteCollection = false">
            <template #title>
                Delete Collection
            </template>
            <template #content>
                Are you sure you want to delete this collection?
            </template>
            <template #footer>
                <jet-secondary-button @click.native="confirmingDeleteCollection = false">
                    Nevermind
                </jet-secondary-button>

                <jet-danger-button class="ml-2" @click.native="deleteCollection" :class="{ 'opacity-25': processing }" :disabled="processing">
                    Delete Collection
                </jet-danger-button>
            </template>
        </jet-confirmation-modal>

        <jet-confirmation-modal :show="userBeingRemoved" @close="userBeingRemoved = null">
            <template #title>
                Remove User
            </template>
            <template #content>
                Are you sure you want to remove this user?
            </template>
            <template #footer>
                <jet-secondary-button @click.native="userBeingRemoved = null">
                    Nevermind
                </jet-secondary-button>

                <jet-danger-button class="ml-2" @click.native="removeUser" :class="{ 'opacity-25': removeUserForm.processing }" :disabled="removeUserForm.processing">
                    Remove User
                </jet-danger-button>
            </template>
        </jet-confirmation-modal>

        <subscription-prompt-modal :show="showingSubscriptionPrompt" @close="showingSubscriptionPrompt = false" />
    </journal-layout>
</template>

<script>
import JournalLayout from '@/Layouts/JournalLayout'
import Bullet from '@/Components/Bullet'
import ContentUpdateNotification from '@/Components/ContentUpdateNotification'
import NewBullet from '@/Components/NewBullet'
import JetConfirmationModal from '@/Jetstream/ConfirmationModal'
import JetDropdown from '@/Jetstream/Dropdown'
import JetDropdownLink from '@/Jetstream/DropdownLink'
import JetSecondaryButton from '@/Jetstream/SecondaryButton'
import JetDangerButton from '@/Jetstream/DangerButton'
import JetInputError from '@/Jetstream/InputError'
import SubscriptionPromptModal from '@/Components/SubscriptionPromptModal'
import Icon from '@/Components/Icon'
import { Link } from '@inertiajs/inertia-vue'
import draggable from 'vuedraggable'

export default {
    components: {
        draggable,
        Bullet,
        ContentUpdateNotification,
        JournalLayout,
        Link,
        NewBullet,
        JetConfirmationModal,
        JetDropdown,
        JetDropdownLink,
        JetSecondaryButton,
        JetDangerButton,
        JetInputError,
        SubscriptionPromptModal,
        Icon,
    },

    props: ['collection'],

    data() {
        return {
            processing: false,
            name: this.collection.name,
            type: this.collection.type,
            hideDone: this.collection.hide_done,
            drawer: false,
            confirmingClearDone: false,
            confirmingDeleteCollection: false,
            showingSubscriptionPrompt: false,

            addUserForm: this.$inertia.form({
                email: '',
            }),

            userBeingRemoved: null,
            removeUserForm: this.$inertia.form(),
            contentUpdateAvailable: false,
            reloading: false,
        }
    },

    watch: {
        name() {
            this.update()
        },
        type() {
            this.update()
        },
        hideDone() {
            this.update()
        }
    },

    mounted() {
        this.listenForUpdates()
        this.reloadWhenHiddenIfContentAvailable()
    },

    methods: {
        listenForUpdates() {
            window.Echo
                .private(`collection.${this.collection.id}`)
                .listen('CollectionUpdated', () => {
                    if (document.visibilityState === 'hidden') {
                        this.reload()
                    } else {
                        this.contentUpdateAvailable = true
                    }
                })

            window.Echo.connector.pusher.connection.bind('reconnected', () => this.reload());

            this.$once('hook:destroyed', () => {
                window.Echo.leave(`collection.${this.collection.id}`)
                window.Echo.connector.pusher.connection.unbind('reconnected')
            })
        },

        reloadWhenHiddenIfContentAvailable() {
            const handler = (e) => {
                if (document.visibilityState === 'hidden' && this.contentUpdateAvailable) {
                    this.reload()
                }
            }

            document.addEventListener('visibilitychange', handler)

            this.$once('hook:destroyed', () => document.removeEventListener('visibilitychange', handler))
        },

        update() {
            this.$inertia.patch(
                route('c.update', this.collection.hashid),
                {
                    name: this.name,
                    type: this.type,
                    hide_done: this.hideDone,
                },
                { preserveScroll: true, headers: { 'X-Socket-ID': Echo.socketId() } }
            )
        },

        storeBullet(bullet) {
            return new Promise((resolve, reject) => {
                this.$inertia.post(
                    route('c.bullets.store', this.collection.hashid),
                    bullet,
                    {
                        preserveScroll: true,
                        headers: { 'X-Socket-ID': Echo.socketId() },
                        onSuccess: () => {
                            if (this.$page.props.showSubscriptionPrompt) {
                                this.showingSubscriptionPrompt = true
                            }
                            resolve()
                        },
                        onError: () => reject(),
                    }
                )
            })
        },

        updateBullet(bullet) {
            this.$inertia.patch(
                route('c.bullets.update', [this.collection.hashid, bullet.id]),
                bullet,
                { preserveScroll: true, headers: { 'X-Socket-ID': Echo.socketId() } }
            )
        },

        migrateBulletTo(bullet, collection) {
            this.$inertia.put(
                route('c.bullets.move', collection.hashid),
                { id: bullet.id },
                { preserveScroll: true, headers: { 'X-Socket-ID': Echo.socketId() } }
            )
        },

        migrateBulletToDailyLog(bullet) {
            this.$inertia.put(
                route('daily-log.move'),
                { id: bullet.id, date: this.$today().format('YYYY-MM-DD') },
                { preserveScroll: true, headers: { 'X-Socket-ID': Echo.socketId() } }
            )
        },

        deleteBullet(bullet) {
            this.$inertia.delete(
                route('c.bullets.destroy', [this.collection.hashid, bullet.id]),
                { preserveScroll: true, headers: { 'X-Socket-ID': Echo.socketId() } }
            )
        },

        clearDone() {
            this.processing = true
            this.$inertia.delete(
                route('c.destroy-done', this.collection.hashid),
                {
                    preserveScroll: true,
                    headers: { 'X-Socket-ID': Echo.socketId() },
                    onFinish: () => {
                        this.processing = false
                        this.confirmingClearDone = false
                    }
                }
            )
        },

        deleteCollection() {
            this.processing = true
            this.$inertia.delete(
                route('c.destroy', this.collection.hashid),
                {
                    preserveScroll: true,
                    headers: { 'X-Socket-ID': Echo.socketId() },
                    onFinish: () => this.processing = false
                }
            )
        },

        addUser() {
            this.processing = true
            this.$inertia.post(
                route('c.users.store', this.collection.hashid),
                { email: this.addUserEmail },
                {
                    preserveScroll: true,
                    headers: { 'X-Socket-ID': Echo.socketId() },
                    onFinish: () => {
                        this.processing = false
                        this.addUserEmail = ''
                    }
                }
            )
        },

        confirmUserRemoval(user) {
            this.userBeingRemoved = user
        },

        removeUser() {
            this.removeUserForm.delete(route('c.users.destroy', [this.collection.hashid, this.userBeingRemoved]), {
                errorBag: 'removeUser',
                preserveScroll: true,
                preserveState: true,
                onSuccess: () => (this.userBeingRemoved = null),
            })
        },

        reload() {
            this.reloading = true

            this.$inertia.reload({
                only: ['collection'],
                headers: { 'X-Socket-ID': Echo.socketId() },
                onFinish: () => {
                    this.contentUpdateAvailable = false
                    this.$nextTick(() => this.reloading = false)
                }
            })
        },

        saveOrder() {
            this.$inertia.put(
                route('c.order.update', this.collection.hashid),
                this.collection.bullets.map(bullet => bullet.id),
                {
                    preserveScroll: true,
                    headers: { 'X-Socket-ID': Echo.socketId() },
                }
            )
        }
    }
}
</script>
