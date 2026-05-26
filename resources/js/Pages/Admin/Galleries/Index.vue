<script setup>
import { ref, watch } from 'vue';
import { Head, Link, useForm, router } from '@inertiajs/vue3';
import AdminLayout from '@/Layouts/AdminLayout.vue';
import Modal from '@/Components/Modal.vue';
import PrimaryButton from '@/Components/PrimaryButton.vue';
import SecondaryButton from '@/Components/SecondaryButton.vue';
import InputLabel from '@/Components/InputLabel.vue';
import TextInput from '@/Components/TextInput.vue';
import InputError from '@/Components/InputError.vue';
import FileUpload from '@/Components/Admin/FileUpload.vue';
import ConfirmModal from '@/Components/Admin/ConfirmModal.vue';
import ToastNotification from '@/Components/Admin/ToastNotification.vue';

const props = defineProps({
    galleries: Array
});

const showCreateModal = ref(false);
const showEditModal = ref(false);
const editingGallery = ref(null);
const showConfirmModal = ref(false);
const galleryToDelete = ref(null);
const toastMessage = ref('');
const toastType = ref('success');

const createForm = useForm({
    album_name: '',
    description: '',
    cover_image: null,
});

const editForm = useForm({
    album_name: '',
    description: '',
    cover_image: null,
});

const openCreateModal = () => {
    createForm.reset();
    createForm.clearErrors();
    showCreateModal.value = true;
};

const closeCreateModal = () => {
    showCreateModal.value = false;
    createForm.reset();
};

const createAlbum = () => {
    createForm.post(route('admin.galleries.store'), {
        onSuccess: () => {
            closeCreateModal();
            showToast('Album berhasil ditambahkan');
        },
    });
};

const openEditModal = (gallery) => {
    editingGallery.value = gallery;
    editForm.album_name = gallery.album_name;
    editForm.description = gallery.description || '';
    editForm.cover_image = null;
    editForm.clearErrors();
    showEditModal.value = true;
};

const closeEditModal = () => {
    showEditModal.value = false;
    editingGallery.value = null;
    editForm.reset();
};

const updateAlbum = () => {
    editForm.post(route('admin.galleries.update', editingGallery.value.id), {
        _method: 'put',
        onSuccess: () => {
            closeEditModal();
            showToast('Album berhasil diperbarui');
        },
    });
};

const confirmDelete = (gallery) => {
    galleryToDelete.value = gallery;
    showConfirmModal.value = true;
};

const deleteAlbum = () => {
    router.delete(route('admin.galleries.destroy', galleryToDelete.value.id), {
        onSuccess: () => {
            showConfirmModal.value = false;
            galleryToDelete.value = null;
            showToast('Album berhasil dihapus');
        }
    });
};

const showToast = (message, type = 'success') => {
    toastMessage.value = message;
    toastType.value = type;
    setTimeout(() => {
        toastMessage.value = '';
    }, 3000);
};
</script>

<template>
    <Head title="Manajemen Galeri" />

    <AdminLayout>
        <template #header>
            <div class="flex justify-between items-center">
                <h2 class="font-semibold text-xl text-gray-800 leading-tight">Manajemen Galeri</h2>
                <PrimaryButton @click="openCreateModal">
                    Tambah Album Baru
                </PrimaryButton>
            </div>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <!-- Grid -->
                <div v-if="galleries.length > 0" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                    <div
                        v-for="gallery in galleries"
                        :key="gallery.id"
                        class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden"
                    >
                        <!-- Cover Image -->
                        <div class="h-48 bg-gray-100">
                            <img
                                v-if="gallery.cover_image_url"
                                :src="gallery.cover_image_url"
                                :alt="gallery.album_name"
                                class="w-full h-full object-cover"
                            />
                            <div v-else class="w-full h-full flex items-center justify-center text-gray-400">
                                <svg class="w-12 h-12" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                                </svg>
                            </div>
                        </div>

                        <!-- Info -->
                        <div class="p-5">
                            <h3 class="text-lg font-semibold text-gray-900 truncate">{{ gallery.album_name }}</h3>
                            <p class="text-sm text-gray-500 mt-1">
                                {{ gallery.photos_count }} foto
                            </p>
                            <p class="text-xs text-gray-400 mt-1">
                                Dibuat {{ new Date(gallery.created_at).toLocaleDateString('id-ID', { day: 'numeric', month: 'long', year: 'numeric' }) }}
                            </p>

                            <!-- Actions -->
                            <div class="flex gap-4 mt-4 pt-4 border-t border-gray-100">
                                <Link
                                    :href="route('admin.galleries.show', gallery.id)"
                                    class="text-sm text-emerald-600 hover:text-emerald-700 font-medium"
                                >
                                    Lihat
                                </Link>
                                <button
                                    @click="openEditModal(gallery)"
                                    class="text-sm text-blue-600 hover:text-blue-700 font-medium"
                                >
                                    Edit
                                </button>
                                <button
                                    @click="confirmDelete(gallery)"
                                    class="text-sm text-red-600 hover:text-red-700 font-medium"
                                >
                                    Hapus
                                </button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Empty State -->
                <div v-else class="bg-white rounded-xl shadow-sm border border-gray-100 p-12 text-center">
                    <svg class="mx-auto h-16 w-16 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                    </svg>
                    <h3 class="mt-4 text-lg font-semibold text-gray-900">Belum ada album galeri</h3>
                    <p class="mt-2 text-sm text-gray-500">Mulai dengan menambahkan album galeri baru.</p>
                    <PrimaryButton class="mt-6" @click="openCreateModal">
                        Tambah Album Baru
                    </PrimaryButton>
                </div>
            </div>
        </div>

        <!-- Create Modal -->
        <Modal :show="showCreateModal" @close="closeCreateModal" maxWidth="lg">
            <div class="p-6">
                <h3 class="text-lg font-bold text-gray-900 mb-6">Tambah Album Baru</h3>

                <form @submit.prevent="createAlbum" class="space-y-6">
                    <div>
                        <InputLabel for="create-album-name" value="Nama Album" />
                        <TextInput
                            id="create-album-name"
                            v-model="createForm.album_name"
                            class="mt-1 block w-full"
                            placeholder="Masukkan nama album"
                        />
                        <InputError :message="createForm.errors.album_name" />
                    </div>

                    <div>
                        <InputLabel for="create-description" value="Deskripsi" />
                        <textarea
                            id="create-description"
                            v-model="createForm.description"
                            rows="3"
                            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                            placeholder="Masukkan deskripsi album (opsional)"
                        ></textarea>
                        <InputError :message="createForm.errors.description" />
                    </div>

                    <div>
                        <FileUpload
                            v-model="createForm.cover_image"
                            label="Cover Image"
                            :error="createForm.errors.cover_image"
                        />
                    </div>

                    <div class="flex justify-end gap-3">
                        <SecondaryButton @click="closeCreateModal" type="button">Batal</SecondaryButton>
                        <PrimaryButton :disabled="createForm.processing" type="submit">Simpan</PrimaryButton>
                    </div>
                </form>
            </div>
        </Modal>

        <!-- Edit Modal -->
        <Modal :show="showEditModal" @close="closeEditModal" maxWidth="lg">
            <div class="p-6">
                <h3 class="text-lg font-bold text-gray-900 mb-6">Edit Album</h3>

                <form @submit.prevent="updateAlbum" class="space-y-6">
                    <div>
                        <InputLabel for="edit-album-name" value="Nama Album" />
                        <TextInput
                            id="edit-album-name"
                            v-model="editForm.album_name"
                            class="mt-1 block w-full"
                            placeholder="Masukkan nama album"
                        />
                        <InputError :message="editForm.errors.album_name" />
                    </div>

                    <div>
                        <InputLabel for="edit-description" value="Deskripsi" />
                        <textarea
                            id="edit-description"
                            v-model="editForm.description"
                            rows="3"
                            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                            placeholder="Masukkan deskripsi album (opsional)"
                        ></textarea>
                        <InputError :message="editForm.errors.description" />
                    </div>

                    <div>
                        <FileUpload
                            v-model="editForm.cover_image"
                            label="Cover Image"
                            :preview-url="editingGallery?.cover_image_url"
                            :error="editForm.errors.cover_image"
                        />
                    </div>

                    <div class="flex justify-end gap-3">
                        <SecondaryButton @click="closeEditModal" type="button">Batal</SecondaryButton>
                        <PrimaryButton :disabled="editForm.processing" type="submit">Simpan</PrimaryButton>
                    </div>
                </form>
            </div>
        </Modal>

        <!-- Delete Confirm Modal -->
        <ConfirmModal
            :show="showConfirmModal"
            title="Hapus Album?"
            :message="`Apakah Anda yakin ingin menghapus album ${galleryToDelete?.album_name}? Semua foto di dalamnya akan ikut terhapus.`"
            confirm-text="Ya, Hapus"
            @confirm="deleteAlbum"
            @close="showConfirmModal = false"
        />

        <!-- Toast Notification -->
        <ToastNotification v-if="toastMessage" :message="toastMessage" :type="toastType" />

        <!-- Flash Success Toast -->
        <ToastNotification
            v-if="$page.props.flash?.success"
            :message="$page.props.flash.success"
            type="success"
        />
    </AdminLayout>
</template>
/home/engine/.bashrc: line 1: syntax error near unexpected token `('
/home/engine/.bashrc: line 1: `. /etc/profile.d/workload-containment.shn# ~/.bashrc: executed by bash(1) for non-login shells.'
/home/engine/.bashrc: line 1: syntax error near unexpected token `('
/home/engine/.bashrc: line 1: `. /etc/profile.d/workload-containment.shn# ~/.bashrc: executed by bash(1) for non-login shells.'
/home/engine/.bashrc: line 1: syntax error near unexpected token `('
/home/engine/.bashrc: line 1: `. /etc/profile.d/workload-containment.shn# ~/.bashrc: executed by bash(1) for non-login shells.'
