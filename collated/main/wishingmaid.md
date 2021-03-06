# wishingmaid
###### \java\seedu\address\logic\commands\AddPhotoCommand.java
``` java
package seedu.address.logic.commands;

import static java.util.Objects.requireNonNull;
import static seedu.address.logic.parser.CliSyntax.PREFIX_FILEPATH;

import java.io.IOException;
import java.util.List;

import seedu.address.commons.core.Messages;
import seedu.address.commons.core.index.Index;
import seedu.address.logic.commands.exceptions.CommandException;
import seedu.address.model.person.Person;
import seedu.address.model.person.Photo;
import seedu.address.model.person.ReadOnlyPerson;
import seedu.address.model.person.exceptions.DuplicatePersonException;
import seedu.address.model.person.exceptions.PersonNotFoundException;
import seedu.address.storage.PhotoStorage;

/**
 * Adds a photo to the specified contact.
 * Incorrect formats of AddPhotoCommand will throw different exceptions.
 * */
public class AddPhotoCommand extends UndoableCommand {
    public static final String COMMAND_WORD = "addphoto";
    public static final String MESSAGE_USAGE = COMMAND_WORD
            + ": adds a photo to the person identified by the index number used in the last person listing.\n"
            + "Parameters: INDEX (must be a positive integer) " + PREFIX_FILEPATH + " (must be valid filepath)\n"
            + "Example: " + COMMAND_WORD + " 1 " + PREFIX_FILEPATH + " C:/users/Desktop/imageFolder/Books";

    public static final String MESSAGE_ADD_PHOTO_SUCCESS = "New photo added to: %1$s";
    public static final String MESSAGE_DELETE_PHOTO_SUCCESS = "Removed photo from Person: %1$s";
    public static final String MESSAGE_NO_PHOTO_TO_DELETE = "This person has no photo to delete.";
    public static final String MESSAGE_DUPLICATE_PERSON = "This person already exists in PEERSONAL.";
    private final Index index;
    private final Photo photo;
    public AddPhotoCommand(Index index, Photo photo) {
        requireNonNull(index);
        requireNonNull(photo);
        this.index = index;
        this.photo = photo;
    }
    @Override
    public CommandResult executeUndoableCommand() throws CommandException {
        List<ReadOnlyPerson> lastShownList = model.getFilteredPersonList();
        if (index.getZeroBased() >= lastShownList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
        }
        ReadOnlyPerson personToEdit = lastShownList.get(index.getZeroBased());
        if (personToEdit.getPhoto().getFilePath().isEmpty() && photo.getFilePath().isEmpty()) {
            throw new CommandException(MESSAGE_NO_PHOTO_TO_DELETE);
        } else if (!photo.getFilePath().equals("")) {
            try {
                //produces a new filepath and rewrites the new filepath to the photo object held by the contact
                PhotoStorage rewrite = new PhotoStorage(photo.getFilePath(), personToEdit.getPhoto().hashCode());
                photo.resetFilePath(rewrite.setNewFilePath());
            } catch (IOException e) {
                throw new CommandException(e.getMessage());
            }
        }
        Person editedPerson = new Person(personToEdit.getName(), personToEdit.getPhone(), personToEdit.getEmail(),
                personToEdit.getAddress(), personToEdit.getRemark(), personToEdit.getBirthday(),
                personToEdit.getAge(), photo,
                personToEdit.getTags());
        try {
            model.updatePerson(personToEdit, editedPerson);
        } catch (DuplicatePersonException dpe) {
            throw new CommandException(MESSAGE_DUPLICATE_PERSON);
        } catch (PersonNotFoundException fpe) {
            throw new AssertionError("The target person cannot be missing.");
        }
        model.getFilteredPersonList();
        return new CommandResult(generateSuccessMessage(editedPerson));
    }
    /**Generate success message*/
    private String generateSuccessMessage(ReadOnlyPerson personToEdit) {
        if (!photo.getFilePath().isEmpty()) {
            return String.format(MESSAGE_ADD_PHOTO_SUCCESS, personToEdit);
        } else {
            return String.format(MESSAGE_DELETE_PHOTO_SUCCESS, personToEdit);
        }
    }



}
```
###### \java\seedu\address\logic\parser\AddPhotoCommandParser.java
``` java
package seedu.address.logic.parser;

import static java.util.Objects.requireNonNull;
import static seedu.address.commons.core.Messages.MESSAGE_INVALID_COMMAND_FORMAT;
import static seedu.address.logic.parser.CliSyntax.PREFIX_FILEPATH;

import seedu.address.commons.core.index.Index;
import seedu.address.commons.exceptions.IllegalValueException;
import seedu.address.logic.commands.AddPhotoCommand;
import seedu.address.logic.parser.exceptions.ParseException;
import seedu.address.model.person.Photo;

/**
 * Parses input arguments and creates a new AddPhotoCommand object
 */
public class AddPhotoCommandParser implements Parser<AddPhotoCommand> {

    /**
     * Parses the given {@code String} of arguments in the context of the AddPhotoCommand
     * and returns an AddPhotoCommand object for execution.
     *
     * @throws ParseException if the user input does not conform the expected format
     */
    public AddPhotoCommand parse(String args) throws ParseException {
        requireNonNull(args);
        ArgumentMultimap argMultimap =
                ArgumentTokenizer.tokenize(args, PREFIX_FILEPATH);
        Index index;
        String filepath;
        try {
            index = ParserUtil.parseIndex(argMultimap.getPreamble());
            filepath = argMultimap.getValue(PREFIX_FILEPATH).orElse("");
            Photo photo = new Photo(filepath);
            return new AddPhotoCommand(index, photo);
        } catch (IllegalValueException ive) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT, AddPhotoCommand.MESSAGE_USAGE));
        } catch (IllegalArgumentException e) {
            throw new ParseException(e.getMessage(), e);
        }
    }
}
```
###### \java\seedu\address\MainApp.java
``` java
    protected InitImageFolder imageFolder;
```
###### \java\seedu\address\MainApp.java
``` java
        logger.info("=============================[ Initializing PEERSONAL ]===========================");
```
###### \java\seedu\address\MainApp.java
``` java
        imageFolder = new InitImageFolder(userPrefs.getDisplayPicturesPath());
```
###### \java\seedu\address\model\person\Photo.java
``` java
package seedu.address.model.person;

import java.io.File;

/**
 * Represents a Photo in the address book.
 */

public class Photo {
    public static final String URL_VALIDATION = "The filepath URL does not exist.";
    private static final String DEFAULT_PHOTOURL = "";
    private static final String DEFAULT_FILEPATH = "";
    private String filepath;
    private String url;
    public Photo(String filepath) throws IllegalArgumentException {
        //this is to setup the default photo for contacts after it is added.
        if (filepath.equals(DEFAULT_FILEPATH)) {
            this.filepath = DEFAULT_PHOTOURL;
            this.url = DEFAULT_FILEPATH;
        } else {
            File file = new File(filepath);
            if (isValidFilePath(file)) {
                this.filepath = filepath;
            } else {
                throw new IllegalArgumentException(URL_VALIDATION);
            }
        }
    }

    public boolean isValidFilePath(File file) {
        return file.exists();
    }
    //the filepath of the image
    public String getFilePath() {
        return filepath;
    }
    //url of the image that is parsed into Image class
    public String getUrl() {
        return this.url;
    }
    /** It is guaranteed that the new filepath exists inside the resources folder */
    public void resetFilePath(String filepath) {
        this.filepath = filepath;
    }
    @Override
    public String toString() {
        return filepath;
    }
    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof Photo // instanceof handles nulls
                && this.filepath.equals(((Photo) other).filepath)); // state check
    }
    @Override
    public int hashCode() {
        return filepath.hashCode();
    }
}
```
###### \java\seedu\address\model\UserPrefs.java
``` java
    private String displayPicturesPath = "displaypictures";
```
###### \java\seedu\address\model\UserPrefs.java
``` java
                && Objects.equals(displayPicturesPath, o.displayPicturesPath);
```
###### \java\seedu\address\model\UserPrefs.java
``` java
        return Objects.hash(guiSettings, addressBookFilePath, addressBookName, displayPicturesPath);
```
###### \java\seedu\address\model\UserPrefs.java
``` java
        sb.append("\nAddressBook image storage filepath : " + displayPicturesPath);
```
###### \java\seedu\address\model\UserPrefs.java
``` java
    public String getDisplayPicturesPath() {
        return displayPicturesPath;
    }
}
```
###### \java\seedu\address\storage\InitImageFolder.java
``` java
package seedu.address.storage;

import static java.util.Objects.requireNonNull;

import java.io.File;
import java.io.IOException;

/**
 * This class creates a image storage folder in the same directory as the addressbook jar file upon running
 * the main app.
 */
public class InitImageFolder {

    public InitImageFolder(String destinationPath) throws IOException {
        requireNonNull(destinationPath);
        File file = new File(destinationPath);
        if (!file.exists()) {
            file.mkdir();
        }
    }
}
```
###### \java\seedu\address\storage\PhotoStorage.java
``` java
package seedu.address.storage;

import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

import javax.imageio.ImageIO;

/**
 * Represents the conversion of a local filepath inside the user's hard drive
 * into a new filepath that is local to the jar file of PEERSONAL.
 * Guarantees : The newly written filepath must exist if ImageIO.write is successful
 */
public class PhotoStorage {
    public static final String WRITE_FAILURE_MESSAGE = "Unable to write to local resource folder: displaypictures. "
            + "Make sure that the image type is supported. Supported types: JPEG, PNG, GIF.";
    private File fileReader = null;
    private String filePath = "";
    private File fileWriter = null;
    private BufferedImage imageReader = null;
    private int uniqueFileName;
    public PhotoStorage(String filePath, int uniqueFileName) {
        this.uniqueFileName = uniqueFileName;
        this.filePath = filePath;
        imageReader = new BufferedImage(300, 400, BufferedImage.TYPE_INT_ARGB);
    }

    public String setNewFilePath() throws IOException {
        String newFilePath = "displaypictures/" + uniqueFileName + ".jpg";
        try {
            fileReader = new File(filePath);
            imageReader = ImageIO.read(fileReader);
            fileWriter = new File(newFilePath);
            ImageIO.write(imageReader, "jpg", fileWriter);
            return newFilePath;
        } catch (IOException e) {
            throw new IOException(WRITE_FAILURE_MESSAGE);
        } catch (IllegalArgumentException f) {
            throw new IOException(WRITE_FAILURE_MESSAGE);
        }
    }
}
```
###### \java\seedu\address\storage\XmlAdaptedPerson.java
``` java
    @XmlElement(required = true)
    private String filepath;
```
###### \java\seedu\address\storage\XmlAdaptedPerson.java
``` java
        filepath = source.getPhoto().getFilePath();
```
###### \java\seedu\address\storage\XmlAdaptedPerson.java
``` java
        final Photo photo = new Photo(this.filepath);
```
###### \java\seedu\address\ui\PersonCard.java
``` java
    private static final String FXML = "PersonListCard.fxml";
    private static String[] colors = { "red", "yellow", "blue", "orange", "brown", "green", "pink", "black", "grey" };
    private static HashMap<String, String> tagColors = new HashMap<String, String>();
    private static Random random = new Random();
```
###### \java\seedu\address\ui\PersonCard.java
``` java
    @FXML
    private ImageView imageView;
```
###### \java\seedu\address\ui\PersonCard.java
``` java
    private static String getColorForTag(String tagValue) {
        if (!tagColors.containsKey(tagValue)) {
            tagColors.put(tagValue, colors[random.nextInt(colors.length)]);
        }
        return tagColors.get(tagValue);
    }
```
###### \java\seedu\address\ui\PersonCard.java
``` java
        setImage(person);
```
###### \java\seedu\address\ui\PersonCard.java
``` java
    /** Changes the tag colour*/
    private void initTags(ReadOnlyPerson person) {
        person.getTags().forEach(tag -> {
            Label tagLabel = new Label(tag.tagName);
            tagLabel.setStyle("-fx-background-color: " + getColorForTag(tag.tagName));
            tags.getChildren().add(tagLabel);
        });
    }
```
###### \java\seedu\address\ui\PersonCard.java
``` java
    /** Checks if the user has added any photo to the specific contact*/
    private void setImage(ReadOnlyPerson person) {
        String url = person.getPhoto().getFilePath(); //gets the filepath directly from the resources folder.
        if (url.equals("")) {
            Image image = new Image(getClass().getResource("/images/noPhoto.png").toExternalForm());
            imageView.setImage(image);
        } else {
            Image image = new Image("file:" + person.getPhoto().getFilePath());
            imageView.setImage(image);
        }
    }
```
