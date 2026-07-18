# Private application development

Private application development is a local platform workflow. It does not use the official Talpaversum Author Registry.

A private developer does not:

- submit an author request,
- receive an `author_id`,
- receive an author certificate,
- use the official author workspace,
- publish to the official Talpaversum catalog.

## Workflow

1. Start Hekatoncheiros Core and Web.
2. Develop the application according to the application and manifest specifications.
3. Start the application backend, for example as a Docker container.
4. Open Developer Tools in Hekatoncheiros Web.
5. Create a local application project.
6. Enter the application origin and manifest or private feed URL.
7. Test connectivity.
8. Review and add the trusted origin.
9. Validate the manifest or feed.
10. Install the application.
11. Verify the application runtime and open the installed application.

Developer Tools must execute this workflow directly. It must not be only a page containing links to Apps Management, Trusted Origins and feed settings.

## Trust

A trusted origin allows Core to retrieve application resources from an explicitly approved origin. It does not create an official author identity and does not make the application trusted by the Talpaversum Author Registry.

A private application must be visibly marked as local or unverified.
