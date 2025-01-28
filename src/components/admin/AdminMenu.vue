<template>
    <nav class="admin-menu">
        <router-link to="/productos/admin" class="admin-link">Productos</router-link>
        <router-link to="/entradasproductos/admin" class="admin-link">Entradas</router-link>
        <router-link to="/salidasproductos/admin" class="admin-link">Salidas</router-link>
        <router-link to="/movimientosinventario/admin" class="admin-link">Movimientos</router-link>
        <router-link to="/inventario/admin" class="admin-link">Inventario</router-link>
        <router-link to="/categoriaproductos/admin" class="admin-link">Categorías</router-link>
        <router-link to="/tamanos/admin" class="admin-link">Tamaños</router-link>
        <router-link to="/longitudes/admin" class="admin-link">Longitudes</router-link>
        <router-link to="/colores/admin" class="admin-link">Colores</router-link>
        <router-link to="/proveedores/admin" class="admin-link">Proveedores</router-link>
        <router-link to="/unidadesmedidas/admin" class="admin-link">Unidades de Medida</router-link>
        <router-link to="/ubicaciones/admin" class="admin-link">Ubicaciones</router-link>
        <router-link to="/usuarios/admin" class="admin-link">Usuarios</router-link>
        <router-link to="/roles/admin" class="admin-link">Roles</router-link>
        <router-link to="/permisos/admin" class="admin-link">Permisos</router-link>
    </nav>
</template>

<style scoped>
.admin-menu {
    position: relative;
    top: 0;
    width: 100%;
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    background-color: #f5f5f5;
    padding: 1rem 2rem;
    justify-content: center;
    align-items: center;
    border-bottom: 1px solid #ddd;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    z-index: 999;
}

.admin-link {
    color: #333;
    text-decoration: none;
    padding: 0.5rem 1rem;
    border-radius: 5px;
    transition: background-color 0.3s ease, color 0.3s ease;
    font-weight: 500;
}

.admin-link:hover {
    background-color: #ff5c5c;
    color: #fff;
}

.router-link-exact-active {
    background-color: #ff5c5c;
    color: #fff;
    font-weight: bold;
    border-radius: 5px;
}
</style>
