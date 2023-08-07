# Complex Example

<Badge type="tip" vertical="top" text="Elementor Pro" /> <Badge type="warning" vertical="top" text="Advanced" />

To showcase a complex form field, we are going create an addon that added a signature field using a 3rd party library.

## Folder Structure

The addon will have two files. One file for the new field-type and the other main file to register the field and handle the registration.

```
elementor-forms-signature-field/
|
├─ form-fields/
|  └─ signature.php
|
└─ elementor-forms-signature-field.php
```

## Plugin Files

**elementor-forms-signature-field.php**

```php
<?php
/**
 * Plugin Name: Elementor Forms Signature Field
 * Description: Custom addon that adds a signature field to Elementor Forms Widget.
 * Plugin URI:  https://elementor.com/
 * Version:     1.0.0
 * Author:      Elementor Developer
 * Author URI:  https://developers.elementor.com/
 * Text Domain: elementor-forms-signature-field
 *
 * Elementor tested up to: 3.7.0
 * Elementor Pro tested up to: 3.7.0
 */

if ( ! defined( 'ABSPATH' ) ) {
	exit; // Exit if accessed directly.
}

/**
 * Register `signature` field-type to Elementor form widget.
 *
 * @since 1.0.0
 * @param \ElementorPro\Modules\Forms\Registrars\Form_Fields_Registrar $form_fields_registrar
 * @return void
 */
function add_new_signature_field( $form_fields_registrar ) {

	require_once( __DIR__ . '/form-fields/signature.php' );

	$form_fields_registrar->register( new \Elementor_Signature_Field() );

}
add_action( 'elementor_pro/forms/fields/register', 'add_new_signature_field' );
```

**form-fields/signature.php**

```php
<?php
if ( ! defined( 'ABSPATH' ) ) {
	exit; // Exit if accessed directly.
}

/**
 * Elementor Form Field - Signature
 *
 * Add a new "Signature" field to Elementor form widget.
 *
 * @since 1.0.0
 */
class Elementor_Signature_Field extends \ElementorPro\Modules\Forms\Fields\Field_Base {

	public $depended_scripts = [ 'signature-pad' ];

	public $depended_styles = [ 'signature-pad' ];

	/**
	 * Get field type.
	 *
	 * Retrieve signature field unique ID.
	 *
	 * @since 1.0.0
	 * @access public
	 * @return string Field type.
	 */
	public function get_type() {
		return 'signature';
	}

	/**
	 * Get field name.
	 *
	 * Retrieve signature field label.
	 *
	 * @since 1.0.0
	 * @access public
	 * @return string Field name.
	 */
	public function get_name() {
		return esc_html__( 'Signature', 'elementor-forms-signature-field' );
	}

	/**
	 * Render field output on the frontend.
	 *
	 * Written in PHP and used to generate the final HTML.
	 *
	 * @since 1.0.0
	 * @access public
	 * @param mixed $item
	 * @param mixed $item_index
	 * @param mixed $form
	 * @return void
	 */
	public function render( $item, $item_index, $form ) {
		$form_id = $form->get_id();
		$unique_id = 'signature_' . $form_id . $item_index;

		$form->add_render_attribute(
			'input' . $item_index,
			[
				'for' => $unique_id,
				'type' => 'hidden',
			]
		);

		$form->add_render_attribute(
			'canvas' . $item_index,
			[
				'for' => $unique_id,
				'class' => 'elementor-signature-field',
			]
		);
		$form->add_render_attribute(
			'clear-button' . $item_index,
			[
				'for' => $unique_id,
				'type' => 'button',
			]
		);
		?>
		<input  <?php echo $form->get_render_attribute_string( 'input' . $item_index ) ?>>
		<canvas <?php echo $form->get_render_attribute_string( 'canvas' . $item_index ) ?>></canvas>
		<button <?php echo $form->get_render_attribute_string( 'clear-button' . $item_index ) ?>>
			<?php echo esc_html__( 'Clear', 'elementor-forms-signature-field' ); ?>
		</button>
		<?php
	}

	/**
	 * Field validation.
	 *
	 * Validate signature field value to ensure it complies to certain rules.
	 *
	 * @since 1.0.0
	 * @access public
	 * @param \ElementorPro\Modules\Forms\Classes\Field_Base   $field
	 * @param \ElementorPro\Modules\Forms\Classes\Form_Record  $record
	 * @param \ElementorPro\Modules\Forms\Classes\Ajax_Handler $ajax_handler
	 * @return void
	 * /
	public function validation( $field, $record, $ajax_handler ) {
		if ( empty( $field['value'] ) ) {
			return;
		}

		if ( preg_match( '/^[0-9]{4}\s[0-9]{4}\s[0-9]{4}\s[0-9]{4}$/', $field['value'] ) !== 1 ) {
			$ajax_handler->add_error(
				$field['id'],
				esc_html__( 'Signature must be in "XXXX XXXX XXXX XXXXX" format.', 'elementor-forms-signature-field' )
			);
		}
	}

	/**
	 * Field constructor.
	 *
	 * Used to add a script to the Elementor editor preview.
	 *
	 * @since 1.0.0
	 * @access public
	 * @return void
	 */
	public function __construct() {
		parent::__construct();
		wp_register_script( 'signature-pad','https://cdn.jsdelivr.net/npm/signature_pad@2.3.2/dist/signature_pad.min.js', [], '2.3.2', true );
		wp_register_style( 'signature-pad', plugins_url( '/assets/signature_pad.css', __FILE__ ), [], '0.1' );

		// add_action( 'elementor/preview/init', [ $this, 'editor_preview_footer' ] );
	}

	/**
	 * Elementor editor preview.
	 *
	 * Add a script to the footer of the editor preview screen.
	 *
	 * @since 1.0.0
	 * @access public
	 * @return void
	 */
	public function editor_preview_footer() {
		// add_action( 'wp_footer', [ $this, 'content_template_script' ] );
	}

	/**
	 * Content template script.
	 *
	 * Add content template alternative, to display the field in Elemntor editor.
	 *
	 * @since 1.0.0
	 * @access public
	 * @return void
	 * /
	public function content_template_script() {
		?>
		<script>
		jQuery( document ).ready( function ( $ ) {
			elementor.hooks.addFilter(
				'elementor_pro/forms/content_template/field/<?php echo $this->get_type(); ?>',
				function ( inputField, item, i ) {
					const fieldType    = 'tel';
					const classes      = 'elementor-field-textual';
					const inputmode    = 'numeric';
					const maxlength    = '19';
					const pattern      = '[0-9\s]{19}';
					const placeholder  = item['signature-placeholder'];
					const autocomplete = 'cc-number';

					return `<input type="${fieldType}" class="${classes}" inputmode="${inputmode}" maxlength="${maxlength}" pattern="${pattern}" placeholder="${placeholder}" autocomplete="${autocomplete}">`;
				}, 10, 3
			);
		} );
		</script>
		<?php
	}
	*/

	public function front_end_inline_JS() {
		$action = __CLASS__ . 'front_end_inline_JS';
		if ( did_action( $action ) ) {
			return;
		}
		do_action( $action );
		?>
		<script>
			jQuery( document ).ready( function( $ ) {
				$canvas = $( 'canvas.elementor-signature-field' );
				$canvas.each( ( index, element ) => {
					const uid = element.id,
						$input = jQuery( 'input[for="' + uid + '"]' ),
						$clearButton = jQuery( 'button[for="' + uid + '"]' ),
						signaturePad = new SignaturePad( element );

					$clearButton.click( () => {
						signaturePad.clear();

						$input.val( '' );
					} );

					$( element ).mouseup( () => {
						const dataURL = signaturePad.toDataURL();

						$input.val( dataURL );
					} );

				} );
			} );
		</script>
		<?php
	}

}
```
