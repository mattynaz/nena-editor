<script>
    import "../app.css";
</script>

<div class="px-8 py-12 min-h-screen flex flex-row ">
    <!-- <aside class="w-48 text-xs relative">
        <p class="absolute bottom-0 italic">
            Matthew Nazari <sup>1,2,3</sup><br>
            matthewnazari@college.harvard.edu<br><br>
            <sup>1</sup>Harvard John A. Paulson School Of Engineering And Applied Sciences<br><br>
            <sup>2</sup>North-Eastern Neo-Aramaic Database Project<br><br>
            <sup>3</sup>Muyšava, ʾUrmi
        </p>
    </aside> -->
    <div class="w-full h-full max-w-screen-sm mx-auto ">
        <slot />
    </div>
    <!-- <aside class="w-48">
    </aside>  -->
</div>
