<script lang="ts">
	import FormWrapperDropdown from "@components/generic/FormWrapperDropdown.svelte"
    import SetNotSetIndicator from "@components/generic/SetNotSetIndicator.svelte"
    import { PROPERTIES } from '@store/stores'
    import { ReceiveNUI } from '@utils/ReceiveNUI'
	import { SendNUI } from '@utils/SendNUI'
	import { SHELLS, TEMP_HIDE } from '@store/stores'
    import { fly } from 'svelte/transition';

    let existingProperties = $PROPERTIES;
    let addingNewProperty = false;

    let description: string = '';
	let for_sale: boolean = true;
	let price: number = 0;
    let period: number = 0;
	let shell: string = Object.keys($SHELLS)[0];
	let door_data: any = null;
	let garage_data: any = null;

    let valid: boolean = false;

    ReceiveNUI('createdDoor', (data) => {
		door_data = data
	});

	ReceiveNUI('createdGarage', (data) => {
		garage_data = data
	});

    function createZone(type) {
        SendNUI('create:createZone', { type: type })
        $TEMP_HIDE = true
    }

    function removeGarage() {
        SendNUI('create:removeGarage', {});
    }

    function createPropertyMethod() {
        SendNUI("create:confirmListing");
        addingNewProperty = false;

        description = '';
        for_sale = true;
        price = 0;
        period = 0;
        shell = Object.keys($SHELLS)[0];
        door_data = null;
        garage_data = null;
    }

    $: {
        valid = description.length > 0 && price > 0 && shell.length > 0 && door_data
        SendNUI('create:setTextFields', {
            description: description,
            for_sale: for_sale,
            price: price,
            period: period,
            shell: shell,
        })
    }
</script>

{#if existingProperties.length <= 0 && !addingNewProperty}
    <div class="no-new-properties-base">
        <img src="images/House.png" alt="House Icon" />

        <p>Anda belum membuat properti baru</p>

        <button on:click={() => addingNewProperty = !addingNewProperty}>
            Buat Properti Baru
        </button>
    </div>
{:else}
    <div class="list-new-property-form">
        <div class="header">
            <div class="heading-title-wrapper">
                <i class="fas fa-circle-plus add-icon"></i>
                <p>Daftar Properti Baru</p>
            </div>
            <div>
                <i class="fas fa-chevron-down chevron-icon"></i>
            </div>
        </div>

        <div class="body-wrapper">
            <div class="left-column">
                <p class="title">Informasi Properti</p>

                <p class="info">Pastikan semua kolom terisi</p>
            </div>
            <div class="right-column">
                <div id="door-creation" class="form-row-wrapper">
                    <p class="label">Buat Pintu</p>
                
                    <div class="action-row">
                        <SetNotSetIndicator leftValue="Pintu" rightValue={door_data ? "Atur" : "Belum Diatur"} good={door_data} />
                        <button class="regular-button" on:click={() => createZone('door')}>{door_data ? 'Belum Diatur' : 'Atur'}</button>
                    </div>
                </div>

                <div id="garage-creation" class="form-row-wrapper">
                    <p class="label">Buat Garasi</p>
                
                    <div class="action-row">
                        <SetNotSetIndicator leftValue="Garasi" rightValue={garage_data ? "Atur" : "Belum Diatur"} good={garage_data} />
                        <button class="regular-button" on:click={() => garage_data ? removeGarage() : createZone('garage')}>{garage_data ? 'Hapus' : 'Atur'}</button>
                    </div>
                </div>

                <div id="description" class="form-row-wrapper">
                    <p class="label">Penjelasan Properti</p>
                
                    <div class="action-row">
                        <textarea rows="5" placeholder="Beri keterangan singkat mengenai properti yang akan dijual..." bind:value={description} />
                    </div>
                </div>

                <div id="price" class="form-row-wrapper">
                    <p class="label">Harga Jual/Sewa</p>
                
                    <div class="action-row">
                        <input type="number" placeholder="1000000000" bind:value={price} />
                    </div>
                </div>

                <div id="period" class="form-row-wrapper">
                    <p class="label">Periode Penarikan (Hari) Khusus Sewa</p>
                
                    <div class="action-row">
                        <input type="number" placeholder="1000000000" bind:value={period} />
                    </div>
                </div>

                <div id="shell-type" class="form-row-wrapper">
                    <p class="label">Tipe Shell</p>
                
                    <div class="action-row">
                        <FormWrapperDropdown dropdownValues={Object.keys($SHELLS)} label="" id="new-listing-dd-shell-type" selectedValue={shell} insideLabel="Tipe: " on:selected-dropdown={(event) => shell = event.detail} />
                    </div>
                </div>
            </div>
        </div>

        <div class="list-new-property-form-footer">
            {#if valid}
                <button on:click={createPropertyMethod}>Buat Properti</button>
            {/if}
        </div>
    </div>
{/if}
