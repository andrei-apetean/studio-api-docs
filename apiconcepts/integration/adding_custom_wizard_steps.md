# Adding custom steps to wizards
<Var:ProductName> Integration API provides support for third-party developers to add custom pages to certain wizards within the <Var:ProductName> desktop application. Custom pages can be added using WPF views and view-models.

*For the moment, only the __Open Package wizard__ supports this functionality, but we may extend it in the future to include other wizards as well.*

Creating pages to use as wizard steps
----
In order to create a view that can be used as a wizard step, a third-party developer needs to:

* Create a WPF view file, e.g. `UserControl`, with custom XAML and/or code behind
* Create a ViewModel for the view, if necessary
* Create a new class representing the custom page
    * This class needs to inherit from the abstract [StudioWizardPage](../../api/integration/Sdl.Desktop.IntegrationApi.Wizard.StudioWizardPage.yml)
    * Implement the required abstract properties

## StudioWizardPage base class
The [StudioWizardPage](../../api/integration/Sdl.Desktop.IntegrationApi.Wizard.StudioWizardPage.yml) abstract class comes with a default implementation for some of the methods and properties required to define a custom wizard page. You can further override these methods and properties to provide your custom logic if needed.

<img style="display:block; margin-left: auto; margin-right: auto;" src="images/Wizard public API.png"/>

## StudioWizardPage properties
Some of the properties are represented on the user interface directly. You can see them in the picture below.
<img style="display:block; " src="images/WizardProperties.png"/>

### ViewType
* Set the `ViewType` property to the actual type of the WPF view file. It will be used to inject the view into the wizard's user interface by <Var:ProductName>'s infrastructure. You can use the `typeof` C# keyword (`GetType` in VB) to obtain the view type needed here.

### ViewModel
* The wizard infrastructure will automatically assign the `ViewModel` as DataContext to the view, so you can use `Binding` from the view to the ViewModel properties. If you don't need a ViewModel just assign `null` here.
* For the `ViewModel` you may choose whatever INotifyPropertyChanged implementation suits you best, such as providing your own or using an MVVM framework.

### IsAvailable
* This property is used by the wizard framework when navigating to other pages to determine if a page should be displayed. 
* When the user clicks **Next** the wizard starts checking through the following pages and displays the first one that is available. The **Next** button is disabled when there are no more pages for which `IsAvailable` returns `true`.
* When the user clicks **Back** the wizard starts checking through the previous pages and displays the first one that is available. The **Back** button is disabled when there are no more pages for which `IsAvailable` returns `true`.

### Data
* You can define custom data for your pages using the `Data` property on the [StudioWizardPage](../../api/integration/Sdl.Desktop.IntegrationApi.Wizard.StudioWizardPage.yml) object.
* The Data property is a `Dictionary<string,object>` that is common to all wizard pages. 
* Once a dictionary key is set from a page, it will be available to any pages that are displayed after it. This includes pages that are navigated to via the Back button. 
* You can use the Page's `OnShow` method to query for the most recent key values that may have been updated/inserted by previously shown pages.
* The `Data` property is initialized for all pages before the wizard is displayed.

### Icon
* Set this property to display an icon in the top-right area of the page.
* This property can be set to `null`. In this case no icon will be displayed.

### Id
* This property is used by the wizard framework to uniquely identify the pages.

  
# Integrating pages as steps into <Var:ProductName> wizards
Custom pages are injected into the wizard via the `firstPages` argument of the event object corresponding to the respective wizard. Keep in mind that the `firstPages` argument is available only for some wizard event objects.
 
To inject the custom pages as steps into a <Var:ProductName> wizard, the third-party developer will require the following steps:

* Determine or define a place in your code where the wizard (with custom steps) would need to be launched; for example, from an [Action](../../api/integration/Sdl.Desktop.IntegrationApi.AbstractAction.yml)'s `Execute` method.
* Determine the appropriate event (e.g. [OpenProjectPackageEvent](../../api/integration/Sdl.TranslationStudioAutomation.IntegrationApi.Events.OpenProjectPackageEvent.yml)) to be raised, that corresponds to the wizard you want to open
* Set up a `List` of [StudioWizardPage](../../api/integration/Sdl.Desktop.IntegrationApi.Wizard.StudioWizardPage.yml) instances indicating the custom pages to be inserted
* Create the event passing in `firstPages` list as an argument
* Use the event aggregator to raise the event. This will launch the wizard.

Notes:
* The custom pages will appear in the wizard at the beginning, before any other standard pages.
* The custom pages will appear in the wizard in the same order they are defined in the list configured in the event object.

Wizard conceptual execution flow
----
The diagram below briefly describes the wizard's execution sequence, and shows the order in which `StudioWizardPage` properties and methods are called by the framework at runtime.

<img style="display:block; " src="images/Wizard public API flow.png"/>

Open package wizard specifics
----
* You can provide your own custom package converter to be used in the Open Package wizard. To do so, create a class that implements the `IExternalPackageConverter` interface and decorate it with the `ExternalPackageConvertor` attribute. The Plugin Framework will pick it up as an extension point.
* In the custom package converter, the wizard `Data` object will be available as well, via the `ExternalPackageConversionInfo.CustomData` property. This way you can use wizard pages to make configurations to your converter if needed.

See also: [full sample application with source code](https://github.com/RWS/trados-studio-api-samples/tree/master/TranslationStudioAutomation/Sdl.CustomWizardSteps.Sample) (on GitHub).

