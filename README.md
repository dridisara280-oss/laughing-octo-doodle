
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
public class PlayerMovement : MonoBehaviour
{
    [Header("Movement")]
    public float walkSpeed = 4f;
    public float sprintSpeed = 7f;
    public float crouchSpeed = 2.5f;
    public float proneSpeed = 1.2f;
    public float jumpHeight = 1.5f;
    public float gravity = -20f;

    [Header("Body")]
    public float standingHeight = 1.8f;
    public float crouchHeight = 1.1f;
    public float proneHeight = 0.55f;
    public float heightChangeSpeed = 5f;

    private CharacterController controller;
    private Vector2 moveInput;
    private Vector3 velocity;

    private bool sprinting;
    private bool crouching;
    private bool prone;

    void Awake()
    {
        controller = GetComponent<CharacterController>();
        controller.height = standingHeight;
        controller.center = Vector3.up * (standingHeight / 2f);
    }

    void Update()
    {
        MovePlayer();
        ChangeBodyHeight();
    }

    // يستقبل حركة عصا التحكم على الهاتف
    public void SetMoveInput(Vector2 input)
    {
        moveInput = Vector2.ClampMagnitude(input, 1f);
    }

    // زر الجري
    public void SetSprinting(bool value)
    {
        sprinting = value;
    }

    // زر القفز
    public void Jump()
    {
        if (controller.isGrounded && !prone)
        {
            velocity.y = Mathf.Sqrt(jumpHeight * -2f * gravity);
        }
    }

    // زر الانحناء
    public void ToggleCrouch()
    {
        crouching = !crouching;

        if (crouching)
        {
            prone = false;
            sprinting = false;
        }
    }

    // زر الاستلقاء
    public void ToggleProne()
    {
        prone = !prone;

        if (prone)
        {
            crouching = false;
            sprinting = false;
        }
    }

    // العودة إلى الوقوف
    public void StandUp()
    {
        crouching = false;
        prone = false;
    }

    void MovePlayer()
    {
        float speed = walkSpeed;

        if (prone)
            speed = proneSpeed;
        else if (crouching)
            speed = crouchSpeed;
        else if (sprinting)
            speed = sprintSpeed;

        Vector3 direction =
            transform.right * moveInput.x +
            transform.forward * moveInput.y;

        controller.Move(direction * speed * Time.deltaTime);

        if (controller.isGrounded && velocity.y < 0)
            velocity.y = -2f;

        velocity.y += gravity * Time.deltaTime;

        controller.Move(velocity * Time.deltaTime);
    }

    void ChangeBodyHeight()
    {
        float targetHeight = standingHeight;

        if (prone)
            targetHeight = proneHeight;
        else if (crouching)
            targetHeight = crouchHeight;

        controller.height = Mathf.Lerp(
            controller.height,
            targetHeight,
            heightChangeSpeed * Time.deltaTime
        );

        controller.center = Vector3.up * (controller.height / 2f);
    }
}
