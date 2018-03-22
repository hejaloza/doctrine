# doctrine
php bin/console doctrine:mapping:convert yml ./src/Entity --from-database --force

php bin/console doctrine:mapping:import AppBundle annotation

php bin/console doctrine:generate:entities AppBundle

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

<?php

namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Method;
use AppBundle\Entity\Producto;
use AppBundle\Entity\Factura;
use AppBundle\Entity\FacturaDetalle;
use DateTime;

class DefaultController extends Controller
{
    /**
     * @Route("/", name="homepage")
     */
    public function indexAction(Request $request)
    {
        // replace this example code with whatever you need
        return $this->render('default/index.html.twig', [
            'base_dir' => realpath($this->getParameter('kernel.project_dir')).DIRECTORY_SEPARATOR,
        ]);
    }
    
        /**
	 * @Route("/listarProductos", name="listarProductos")
	 */
	public function listarProductosAction()
	{
		$repository = $this->getDoctrine()->getRepository('AppBundle:Producto');
		$productos = $repository->findAll();
                
		return $this->render('AppBundle:Producto:listar_productos.html.twig', array('productos' => $productos));
	}
        
        
    /**
     * @Route("/crearProducto", name="crearProducto")
     * @Method({"GET"})
     */
    public function formCrearProductoAction()
    {
    	
    	return $this->render('AppBundle:Producto:crear_producto.html.twig', 
    			array('errors' => [])
    			);
    }     
    
    
    
    /**
     * @Route("/crearProducto", name="crearProductoPost")
     * @Method({"POST"})
     */
    public function crearProductoAction(Request $request)
    {
    	$nombre=$request->request->get('nombre');
    	$descripcion=$request->request->get('descripcion');
    	$tipo_prod=$request->request->get('tipo_prod');
        
        $cantidad=$request->request->get('cantidad');
        $precio_unitario=$request->request->get('precio_unitario');
        $subtotal=$request->request->get('subtotal');
        $iva=$request->request->get('iva');
        $total=$request->request->get('total');
        
        $em = $this->getDoctrine()->getManager();
        $repository = $this->getDoctrine()->getRepository('AppBundle:TipoProducto');
    	$objTipoProducto = $repository->find($tipo_prod);
    	
    	$producto = new Producto();
    	$producto->setNombre($nombre);
    	$producto->setDescripcion($descripcion);
    	$producto->setCodTipoProducto($objTipoProducto);
        $startDate = date("Y-m-d");
        $dateTime=$startDate." 00:00:00";
       // $date = \DateTime::createFromFormat($dateTime)->format("Y-m-d H:i:s");
      //  $date =date_create_from_format('Y-m-d H:i:s', $dateTime);
        $producto->setFechaIngreso(new DateTime ( $dateTime ));
    	
        $factura = new Factura();
    	$factura->setSubtotal($subtotal);
    	$factura->setIva($iva);
    	$factura->setTotal($total);
        
        $em->persist($producto);
        $em->persist($factura);
        $em->flush();
        
        $facturaDetalle = new FacturaDetalle();
    	$facturaDetalle->setSubtotal($subtotal);
    	$facturaDetalle->setCodProducto($producto);
    	$facturaDetalle->setCantidad($cantidad);
        $facturaDetalle->setPrecioUnitario($precio_unitario);
        
        $em->persist($facturaDetalle);
        $em->flush();
    	
    	
    	
    	$this->get('session')->getFlashBag()->set('succesfull', 'Empleado Creado');
    	return $this->redirectToRoute('listarProductos');	
    }
    
    
    
    /**
     * @Route("/editarProducto/{id}", name="editarProductoGet")
     * @Method({"GET"})
     */
    public function editarGetAction($id)
    {
    	
    	$repository = $this->getDoctrine()->getRepository('AppBundle:Producto');
    	$producto = $repository->find($id);	
    	return $this->render('AppBundle:Producto:editar_producto.html.twig', array('producto' => $producto, 'errors' => []));
    	
    }
    
    
    
   /**
     * @Route("/editarProducto/{id}", name="editarProductoPost")
     * @Method({"POST"})
     */
    public function editarPostAction($id,Request $request)
    {
    	$nombre=$request->request->get('nombre');
    	$descripcion=$request->request->get('descripcion');
    	$tipo_prod=$request->request->get('tipo_prod');
    	
    	$repository = $this->getDoctrine()->getRepository('AppBundle:Producto');
    	$producto = $repository->find($id);

    	$producto->setNombre($nombre);
    	$producto->setDescripcion($descripcion);
        $repositoryProd = $this->getDoctrine()->getRepository('AppBundle:TipoProducto');
    	$objTipoProducto = $repositoryProd->find($tipo_prod);
    	$producto->setCodTipoProducto($objTipoProducto);
    	
    	$em = $this->getDoctrine()->getManager();
    	$em->persist($producto);
    	$em->flush();
    	$this->get('session')->getFlashBag()->set('succesfull', 'Cadena Modificada');
    	return $this->redirectToRoute('listarProductos');

    }
    
    
    
}
*****************************************************************************************************************************************

///////////////Listar////////////////

{% extends 'base.html.twig' %}

{% block body %}


         <div class="col-sm-12 col-md-10 col-md-offset-1 main">
          <a class="btn btn-warning" href="{{ path('crearProducto')}}">Crear Producto</a>
         <hr>
          <h2 class="sub-header">Lista de Productos</h2>

          {% for flashMessage in app.session.flashbag.get('succesfull') %}
                <div class='alert alert-success'>
                    {{ flashMessage }}
                </div>
          {% endfor %}
          
          <div class="table-responsive">
            <table class="table table-striped">
                <tr>
                    <th>Nombre</th>
                    <th>Tipo Producto</th>
                    <th>Descripcion</th>
                    <th>Fecha Ingreso</th>
                    
                    <th></th>
                </tr>
                {% for producto in productos %}
                <tr>
                    <td>{{ producto.nombre }}</td>
                    <td>{{ producto.codTipoProducto.nombre }}</td>
                    <td>{{ producto.descripcion }}</td>
                    <td>{{ producto.fechaIngreso|date('Y-m-d') }}</td>
                    
                    <td>
                        <a class="btn btn-primary" href="{{ path('editarProductoGet', {'id' : producto.codigo} )}}">Modificar
                        
                    </td>
                    
                </tr>
                {% endfor %}
            </table>
          </div>
        </div>

{% endblock %}
*******************************************************************************************************************************************
//////////////////////////////editar////

{% extends 'base.html.twig' %}

{% block body %}

<div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-4 main">
        <div class="col-lg-4">
         <a class="btn btn-warning" href="{{ path('listarProductos')}}">Listar Empleados</a>
         <hr>
          <h2 class="sub-header">Editar Producto</h2>
            <form action="" method="POST">
              <div class="form-group">
                <label for="nombre">Nombre:</label>
                <input type="text" class="form-control" id="nombre" value="{{producto.nombre }}" name="nombre" required>
              </div>
              <div class="form-group">
                <label for="descripcion">Descripcion:</label>
                <input type="text" class="form-control" id="descripcion" value="{{producto.descripcion }}" name="descripcion" required>
              </div>
               <div class="form-group">
                <label for="descripcion">Tipo Producto:</label>
                <select class="selectpicker" id="tipo_prod" name="tipo_prod">
                    <option value="1">Limpieza</option>
                    <option value="2">Utiles</option>
                   
                </select>
              </div>
              <button type="submit" class="btn btn-default btn-success">Guardar</button>
            </form>
            <br>
        </div>          
</div>

<div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">
        <div class="col-lg-4">
            {% if errors|length > 0 %}
                <div class="alert alert-danger">
                        <ul>
                        {% for error in errors %}
                           <li>{{ error }}</li>
                        {% endfor %}
                    </ul>
                </div>
            {% endif %}
        </div>  

</div>


{% endblock %}

////////////////////////////////////////////////////////*************************************************************************
******Crear**************************

{% extends 'base.html.twig' %}

{% block body %}

<div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-4 main">
        <div class="col-lg-4">
          <h2 class="sub-header">Crea Producto</h2>
            <a class="btn btn-warning" href="{{ path('listarProductos')}}">Listar Empleados</a>
            <form action="{{ path('crearProducto')}}" method="POST">
              <div class="form-group">
                <label for="nombre">Nombre:</label>
                <input type="text" class="form-control" id="nombre" name="nombre" required>
              </div>
              <div class="form-group">
                <label for="descripcion">Descripcion:</label>
                <input type="text" class="form-control" id="descripcion" name="descripcion" required>
              </div>
               <div class="form-group">
                <label for="descripcion">Tipo Producto:</label>
                <select class="selectpicker" id="tipo_prod" name="tipo_prod">
                    <option value="1">Limpieza</option>
                   
                </select>
              </div>
                <div class="form-group">
                <label for="descripcion">Cantidad:</label>
                <input type="text" class="form-control" id="cantidad" name="cantidad" required>
              </div>
                <div class="form-group">
                <label for="descripcion">Precio Unitario:</label>
                <input type="text" class="form-control" id="precio_unitario" name="precio_unitario" required>
              </div>
                <div class="form-group">
                <label for="descripcion">Subtotal:</label>
                <input type="text" class="form-control" id="subtotal" name="subtotal" required>
              </div>
               <div class="form-group">
                <label for="descripcion">Iva:</label>
                <input type="text" class="form-control" id="iva" name="iva" required>
              </div>
                 <div class="form-group">
                <label for="descripcion">Total:</label>
                <input type="text" class="form-control" id="total" name="total" required>
              </div>
              
              <button type="submit" class="btn btn-default btn-success">Guardar</button>
            </form>
            <br>
        </div>          
</div>

<div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">
        <div class="col-lg-4">
            {% if errors|length > 0 %}
                <div class="alert alert-danger">
                        <ul>
                        {% for error in errors %}
                           <li>{{ error }}</li>
                        {% endfor %}
                    </ul>
                </div>
            {% endif %}
        </div>  

</div>

{% block javascripts %}
       
        
{% endblock %}

{% endblock %}


