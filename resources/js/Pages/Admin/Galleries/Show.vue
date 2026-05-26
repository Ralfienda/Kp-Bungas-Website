<script setup>
import { ref } from 'vue';
import { Head, Link, router, useForm } from '@inertiajs/vue3';
import AdminLayout from '@/Layouts/AdminLayout.vue';
import PrimaryButton from '@/Components/PrimaryButton.vue';
import InputError from '@/Components/InputError.vue';
import ConfirmModal from '@/Components/Admin/ConfirmModal.vue';
import ToastNotification from '@/Components/Admin/ToastNotification.vue';

const props = defineProps({
    gallery: Object
});

const uploadForm = useForm({
    photos: [],
});

const photoPreviews = ref([]);
const showConfirmModal = ref(false);
const photoToDelete = ref(null);
const toastMessage = ref('');
const toastType = ref('success');
const isUploading = ref(false);

const handleFileSelect = (e) => {
    const files = Array.from(e.target.files);
    uploadForm.photos = files;
    
    // Generate previews
    photoPreviews.value = files.map(file => URL.createObjectURL(file));
};

const uploadPhotos = () => {
    if (uploadForm.photos.length === 0) return;
    
    isUploading.value = true;
    uploadForm.post(route('admin.galleries.photos.upload', props.gallery.id), {
        onSuccess: () => {
            uploadForm.reset();
            photoPreviews.value = [];
            isUploading.value = false;
            showToast('Foto berhasil diupload');
        },
        onError: () => {
            isUploading.value = false;
        }
    });
};

const confirmDeletePhoto = (photo) => {
    photoToDelete.value = photo;
    showConfirmModal.value = true;
};

const deletePhoto = () => {
    router.delete(route('admin.galleries.photos.destroy', photoToDelete.value.id), {
        onSuccess: () => {
            showConfirmModal.value = false;
            photoToDelete.value = null;
            showToast('Foto berhasil dihapus');
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
    <Head :title="`Galeri: ${gallery.album_name}`" />

    <AdminLayout>
        <template #header>
            <div class="flex justify-between items-center">
                <div>
                    <Link
                        :href="route('admin.galleries.index')"
                        class="text-sm text-gray-500 hover:text-gray-700 mb-1 inline-block"
                    >
                        &larr; Kembali ke Galeri
                    </Link>
                    <h2 class="font-semibold text-xl text-gray-800 leading-tight">{{ gallery.album_name }}</h2>
                </div>
            </div>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8 space-y-8">
                <!-- Album Info -->
                <div class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
                    <div class="p-6">
                        <div class="flex flex-col md:flex-row gap-6">
                            <!-- Cover -->
                            <div v-if="gallery.cover_image_url" class="md:w-64 shrink-0">
                                <img
                                    :src="gallery.cover_image_url"
                                    :alt="gallery.album_name"
                                    class="w-full h-40 object-cover rounded-lg"
                                />
                            </div>

                            <!-- Info -->
                            <div class="flex-1">
                                <p class="text-gray-700 text-sm leading-relaxed" v-if="gallery.description">
                                    {{ gallery.description }}
                                </p>
                                <p class="text-gray-400 text-sm" v-else>
                                    Tidak ada deskripsi.
                                </p>
                                <p class="text-xs text-gray-400 mt-2">
                                    {{ gallery.photos?.length || 0 }} foto &middot;
                                    Dibuat {{ new Date(gallery.created_at).toLocaleDateString('id-ID', { day: 'numeric', month: 'long', year: 'numeric' }) }}
                                </p>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Upload Section -->
                <div class="bg-white rounded-xl shadow-sm border border-gray-100 p-6">
                    <h3 class="text-lg font-semibold text-gray-900 mb-4">Upload Foto</h3>

                    <form @submit.prevent="uploadPhotos" class="space-y-4">
                        <div class="flex items-center gap-4 flex-wrap">
                            <label
                                class="relative cursor-pointer bg-white rounded-lg border border-gray-300 px-4 py-2 text-sm font-medium text-gray-700 hover:bg-gray-50 focus-within:outline-none focus-within:ring-2 focus-within:ring-indigo-500"
                            >
                                <span>Pilih Foto</span>
                                <input
                                    type="file"
                                    multiple
                                    accept="image/*"
                                    class="sr-only"
                                    @change="handleFileSelect"
                                />
                            </label>

                            <span v-if="uploadForm.photos.length > 0" class="text-sm text-gray-500">
                                {{ uploadForm.photos.length }} file dipilih
                            </span>

                            <PrimaryButton
                                v-if="uploadForm.photos.length > 0"
                                type="submit"
                                :disabled="uploadForm.processing"
                            >
                                {{ uploadForm.processing ? 'Mengupload...' : 'Upload' }}
                            </PrimaryButton>
                        </div>

                        <InputError :message="uploadForm.errors.photos" class="mt-2" />
                        <InputError :message="uploadForm.errors['photos.0']" class="mt-2" />

                        <!-- Preview uploaded photos -->
                        <div v-if="photoPreviews.length > 0" class="grid grid-cols-3 sm:grid-cols-4 md:grid-cols-6 gap-3 mt-4">
                            <div
                                v-for="(preview, index) in photoPreviews"
                                :key="index"
                                class="relative aspect-square rounded-lg overflow-hidden border border-gray-200"
                            >
                                <img
                                    :src="preview"
                                    :alt="`Preview ${index + 1}`"
                                    class="w-full h-full object-cover"
                                />
                            </div>
                        </div>
                    </form>
                </div>

                <!-- Photo Grid -->
                <div class="bg-white rounded-xl shadow-sm border border-gray-100 p-6">
                    <h3 class="text-lg font-semibold text-gray-900 mb-4">Semua Foto</h3>

                    <div v-if="gallery.photos && gallery.photos.length > 0" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-4">
                        <div
                            v-for="photo in gallery.photos"
                            :key="photo.id"
                            class="relative group aspect-square rounded-lg overflow-hidden border border-gray-200"
                        >
                            <img
                                :src="photo.image_url"
                                :alt="photo.caption || 'Foto galeri'"
                                class="w-full h-full object-cover"
                            />

                            <!-- Caption overlay -->
                            <div
                                v-if="photo.caption"
                                class="absolute bottom-0 left-0 right-0 bg-gradient-to-t from-black/60 to-transparent p-2"
                            >
                                <p class="text-xs text-white truncate">{{ photo.caption }}</p>
                            </div>

                            <!-- Delete button -->
                            <button
                                @click="confirmDeletePhoto(photo)"
                                class="absolute top-2 right-2 bg-red-600 text-white rounded-full p-1.5 shadow-md opacity-0 group-hover:opacity-100 transition-opacity hover:bg-red-700"
                                title="Hapus foto"
                            >
                                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                                </svg>
                            </button>
                        </div>
                    </div>

                    <!-- Empty State -->
                    <div v-else class="text-center py-12">
                        <svg class="mx-auto h-16 w-16 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                        </svg>
                        <h3 class="mt-4 text-lg font-semibold text-gray-900">Belum ada foto</h3>
                        <p class="mt-2 text-sm text-gray-500">Upload foto pertama Anda!</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- Delete Photo Confirm Modal -->
        <ConfirmModal
            :show="showConfirmModal"
            title="Hapus Foto?"
            message="Apakah Anda yakin ingin menghapus foto ini?"
            confirm-text="Ya, Hapus"
            @confirm="deletePhoto"
            @close="showConfirmModal = false"
        />

        <!-- Toast Notification -->
        <ToastNotification v-if="toastMessage" :message="toastMessage" :type="toastType" />

        <!-- Flash Toast -->
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
/home/engine/.bashrc: line 1: syntax error near unexpected token `('
/home/engine/.bashrc: line 1: `. /etc/profile.d/workload-containment.shn# ~/.bashrc: executed by bash(1) for non-login shells.'
/home/engine/.bashrc: line 1: syntax error near unexpected token `('
/home/engine/.bashrc: line 1: `. /etc/profile.d/workload-containment.shn# ~/.bashrc: executed by bash(1) for non-login shells.'
/home/engine/.bashrc: line 1: syntax error near unexpected token `('
/home/engine/.bashrc: line 1: `. /etc/profile.d/workload-containment.shn# ~/.bashrc: executed by bash(1) for non-login shells.'
