.SUFFIXES:
.DELETE_ON_ERROR:

.PHONY: clean tests unittest endtoendtest updatestandard checkversions

PYTEST_BIN=python3 -m pytest

PYTHONPATH_PREFIX=PYTHONPATH=..

VERSION=current

cleaned_dicom_html=$(patsubst standard/%.html,tmp/%.html,$(wildcard standard/*.html))


all: core_tables relationship_tables dist/references.json

core_tables: dist/ciods.json dist/modules.json dist/macros.json dist/attributes.json dist/sops.json dist/confidentiality_profile_attributes.json

relationship_tables: dist/ciod_to_modules.json dist/ciod_to_func_group_macros.json dist/module_to_attributes.json dist/macro_to_attributes.json


dist/ciods.json: tmp/raw_ciod_module_tables.json
	$(PYTHONPATH_PREFIX) python3 process_ciods.py $< > $@

dist/modules.json: tmp/preprocessed_modules_attributes.json dist/ciod_to_func_group_macros.json
	$(PYTHONPATH_PREFIX) python3 process_modules.py $^ > $@

dist/macros.json: tmp/preprocessed_macros_attributes.json
	$(PYTHONPATH_PREFIX) python3 process_macros.py $< > $@

dist/attributes.json: tmp/part06.html
	$(PYTHONPATH_PREFIX) python3 extract_attributes.py $< > $@

dist/sops.json: tmp/part04.html
	$(PYTHONPATH_PREFIX) python3 extract_sops.py $< > $@

dist/confidentiality_profile_attributes.json: tmp/part15.html dist/attributes.json
	$(PYTHONPATH_PREFIX) python3 extract_conf_profile_attributes.py $^ > $@

dist/ciod_to_modules.json: tmp/raw_ciod_module_tables.json
	$(PYTHONPATH_PREFIX) python3 process_ciod_module_relationship.py $< > $@

dist/ciod_to_func_group_macros.json: tmp/raw_ciod_func_group_macro_tables.json
	$(PYTHONPATH_PREFIX) python3 process_ciod_func_group_macro_relationship.py $< > $@

dist/module_to_attributes.json: tmp/module_to_attributes_no_duplicates.json dist/macros.json dist/ciod_to_func_group_macros.json dist/macro_to_attributes.json
	$(PYTHONPATH_PREFIX) python3 postprocess_integrate_func_group_macros.py $^ > $@

dist/macro_to_attributes.json: tmp/macros_attributes_updated_references.json
	$(PYTHONPATH_PREFIX) python3 postprocess_merge_duplicate_nodes.py $< > $@

dist/references.json: tmp/modules_attributes_partial_references.json tmp/macros_attributes_partial_references.json tmp/raw_section_tables.json
	$(PYTHONPATH_PREFIX) python3 postprocess_save_references.py $^ > $@


tmp/module_to_attributes_no_duplicates.json: tmp/modules_attributes_updated_references.json
	$(PYTHONPATH_PREFIX) python3 postprocess_merge_duplicate_nodes.py $< > $@

tmp/modules_attributes_updated_references.json: tmp/modules_attributes_partial_references.json dist/references.json
	$(PYTHONPATH_PREFIX) python3 postprocess_update_reference_links.py $^ > $@

tmp/macros_attributes_updated_references.json: tmp/macros_attributes_partial_references.json dist/references.json
	$(PYTHONPATH_PREFIX) python3 postprocess_update_reference_links.py $^ > $@

tmp/modules_attributes_partial_references.json: tmp/modules_attributes_no_references.json
	$(PYTHONPATH_PREFIX) python3 postprocess_mark_references.py $< > $@

tmp/macros_attributes_partial_references.json: tmp/macros_attributes_no_references.json
	$(PYTHONPATH_PREFIX) python3 postprocess_mark_references.py $< > $@

tmp/modules_attributes_no_references.json: tmp/preprocessed_modules_attributes.json
	$(PYTHONPATH_PREFIX) python3 process_module_attribute_relationship.py $< > $@

tmp/macros_attributes_no_references.json: tmp/preprocessed_macros_attributes.json
	$(PYTHONPATH_PREFIX) python3 process_macro_attribute_relationship.py $< > $@

tmp/preprocessed_modules_attributes.json: tmp/raw_module_macro_attribute_tables.json
	$(PYTHONPATH_PREFIX) python3 preprocess_modules_with_attributes.py $< > $@

tmp/preprocessed_macros_attributes.json: tmp/raw_module_macro_attribute_tables.json
	$(PYTHONPATH_PREFIX) python3 preprocess_macros_with_attributes.py $< > $@

tmp/raw_ciod_module_tables.json: tmp/part03.html
	$(PYTHONPATH_PREFIX) python3 extract_ciod_module_tables.py $< > $@

tmp/raw_ciod_func_group_macro_tables.json: tmp/part03.html
	$(PYTHONPATH_PREFIX) python3 extract_ciod_func_group_macro_tables.py $< > $@

tmp/raw_module_macro_attribute_tables.json: tmp/part03.html
	$(PYTHONPATH_PREFIX) python3 extract_modules_macros_with_attributes.py $< > $@

tmp/raw_section_tables.json: extract_sections.py $(cleaned_dicom_html)
	$(PYTHONPATH_PREFIX) python3 $^ > $@


tmp/%.html: standard/%.html
	cat $< | sed -e 's/&nbps;/ /g' -e 's/​//g' > $@


tests: unittest endtoendtest

unittest:
	$(PYTEST_BIN) -m 'not endtoend'

endtoendtest:
	$(PYTEST_BIN) -m 'endtoend'


updatestandard:
	if [ ! -d old_standard ]; then mkdir old_standard; fi
	mv -f standard/*.html old_standard/
	wget --compression=auto http://dicom.nema.org/medical/dicom/${VERSION}/output/html/part03.html -O standard/part03.html
	wget --compression=auto http://dicom.nema.org/medical/dicom/${VERSION}/output/html/part04.html -O standard/part04.html
	wget --compression=auto http://dicom.nema.org/medical/dicom/${VERSION}/output/html/part06.html -O standard/part06.html
	wget --compression=auto http://dicom.nema.org/medical/dicom/${VERSION}/output/html/part15.html -O standard/part15.html
	wget --compression=auto http://dicom.nema.org/medical/dicom/${VERSION}/output/html/part16.html -O standard/part16.html
	wget --compression=auto http://dicom.nema.org/medical/dicom/${VERSION}/output/html/part17.html -O standard/part17.html
	wget --compression=auto http://dicom.nema.org/medical/dicom/${VERSION}/output/html/part18.html -O standard/part18.html

checkversions:
	@python3 --version 2>&1 | grep -q 3.5. || { echo "Need Python 3.5" && exit 1; }

clean:
	git clean -fqx dist tmp
	find . -type f -name "*.py[co]" -delete
	find . -type d -name "__pycache__" -delete
